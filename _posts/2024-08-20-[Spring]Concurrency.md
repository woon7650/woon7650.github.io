---
title: "Solution of Concurrency Issue With Lock"
excerpt: "[Spring] Lock을 통한 동시성 이슈 해결방법"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2024-08-20
---

#### 0. 들어가면서

  - 이직 준비를 하면서 면접에서 자주 등장하는 Backend CS 관련 지식을 정리하고 자세히 알아보고자 합니다.
  - Redis에 대해서 공부하고 토이프로젝트를 준비하면서 기존에 사용했던 RDB위주의 Lock 사용외에 NoSQL Lock에 대해서도 깊이있게 학습해보고자 합니다.

  - ##### Issue(MultiThread)
    - Race Condition
      - Resource에 대해 여러 프로세스가 동시에 접근을 시도할 때 결과값에 영향을 줌
    - DeadLock
      - 두 개 이상의 Thread가 서로의 Lock을 기다리다 아무 Thread도 실행되지 않음

  - ##### Solution

    - 하나의 Thread 작업이 완료된 후에 다른 Thread가 공유 Resource에 접근 가능하도록 해야 함

    - Synchronized
    - RDB Lock
      - Pessimistic Locking
      - Optimistic Locking
      - Named Locking
    - NoSQL(Redis) Lock
      - Lettuce Lock
      - Redisson Lock


<br />

---

### 1. Synchronized

  - Thread 동기화를 위해 자바에서 지원하는 기술
  - 공유자원에 동시에 Thread가 접근하지 못하도록 막음
  
  - ##### 1.1 @Transactional

    - Spring Transaction은 프록시 객체가 실제 객체를 감싸고 있는 형태로 되어 동작함
    - synchronzied 메소드 자체의 동시성은 보장되지만 데이터 업데이트 시점까지 다른 Thread가 데이터를 조회하게 되면 동시성을 보장할 수 없음
      - DB로 Flush하는 시점과 다음 Thread가 해당 메소드를 시작하는 시점이 불일치
      - Flush 시점을 정확히 알 수 없음
      - DB에 반영되지 않은 값을 읽어오는 상황 발생
    - @Transacional과 synchronized는 별개로 사용하거나 synchronzied 안에 @Transactional 메소드를 호출해서 사용

  - ##### 1.2 Multi Server(Scale out)

    - synchronzied는 '동일 Process의 Thread'에 대해서만 동시성을 보장함
    - 여러 대의 서버에서 요청이 들어오는 경우는 동시성을 보장할 수 없음

  - #### 1.3 Code

    ```java
    //@Transactional
    public synchronzied void increase(){
      //Logic
    }
    ```
<br />

---

### 2. RDB Lock

  - #### 2.1 Pessimistic Lock

    - DB에서 데이터에 접근할 떄 배타락을 걸어 다른 Process 에서 데이터에 접근할 수 없도록 하는 방식
    - 일반적으로 PESSIMISTIC_WRITE 사용

      
      ```java
      @Lock(LockModeType.PESSIMITIC_WRITE)
      ```

    - ##### 2.1.1 Pros & Cons

      - Pros
        - 데이터이 정합성이 보장됨(Scale out된 여러 서버에서도 유지)
      - Cons
        - 항상 배타락을 걸고 사용하기 때문에 수행 시간이 늘어남
        - 성능 감소가 일어남
        - DeadLock이 발생할 수 있음


  - #### 2.2 Optimistic Lock

    - Lock을 직접 이용하지 않고 Version Column을 사용해서 읽어온 시점과 Transaction이 커밋되는 시점의 데이터가 같은지 정합성을 비교하는 방식
    - Lock을 Application Level에서 해결
    - update시 version값에 해당하는 데이터만 update, 동일한 update 쿼리가 반복되는 것을 방지
    - version 정보가 틀려서 update 쿼리가 실패하면 재시도

      
      ```java
      @Version
      private Long version;
      ```
    
      ```java
      @Lock(LockModeType.OPTIMISTIC)
      ```

    - ##### 2.2.1 Pros & Cons

      - Pros
        - 데이터이 정합성이 보장됨(Scale out된 여러 서버에서도 유지)
        - 충돌이 없는 경우 Lock을 별도로 잡지 않아서 성능이 좋음
      - Cons
        - Exception 발생 시 Rollback 로직을 Custom해야 함
        - Exception 발생 시 로직을 처음부터 재시도하기 때문에 느림
          - 재귀 방법으로 수행시 Stack Over Flow 발생 가능성이 있음 

    - ##### 2.2.2 Pessimistic Lock

      - 충돌이 빈번하게 발생할 경우
        - 성능 : Pessimistic Lock >> Optimistic Lock

  - #### 2.3 Named Lock

    - 이름을 가진 Metadata Locking으로 이름과 timeout 정보를 얻어서 Lock을 획득하고 해제하는 방식
    - 별도의 접근 공간에 Lock을 걸음
    - Transaction 종료 시에 Lock 자동 해제가 되지 않기 때문에 별도의 명령이 필요(releaseLock)

    - ##### 2.3.1 Pros & Cons

      - Pros
        - Time out을 손쉽게 구현할 수 있음
      - Cons
        - Lock 해제와 Session 관리를 해야함
        - 구현방법이 다소 복잡


    - ##### 2.3.2 Code

      ```java
      public interface LockRepository extends JpaRepository<Member, Long> {
        
        @Query(value = "SELECT get_lock(:key, 3000)", nativeQuery = true)
        void getLock(String key);

        @Query(value = "SELECT release_lock(:key)", nativeQuery = true)
        void releaseLock(String key);
      }
      ```

      ```java
      @Transactional
      public void decrease(Long id, Long quantity) {
        try {
          lockRepository.getLock(id.toString());
          memberRepository.decreaseInNamedLock(id, quantity);
        } finally {
          lockRepository.releaseLock(id.toString());
        }
      }
      ```


<br />

---


### 3. NoSQL(Redis) Lock

  - Redis Client Library
    - Lettuce
    - Redisson
  - 분산락
    - Service가 여러 대인 상황에서 데이터의동기화를 보장하기 위한 Lock 방식
    - Redis는 분산형 메모리 내 데이터 저장소로 분산환경에서의 Lock 구현에 적합
    - Redis는 RDB보다 빠른 응답시간 제공

  - #### 3.1 Lettuce(Spring Data Redis)

    - Spin Lock 방식
    - 재시도가 필요하지 않은 경우(실무)
    
    - lock-unlock & spin lock
      - lock
        - setIfAbsent() : setnx 명령어
          - return
            - 0 : key-value쌍이 존재하지 않고 등록에 성공
            - 1 : 기존에 등록한 키가 이미 존재
        - 인자로 받은 key와 "lock" 으로 key-value쌍 저장을 시도
      - unlock
        - 인자로 받은 key값의 key-value쌍을 메모리에서 삭제
      - spin lock
        - while문

    - ##### 3.1.1 SETNX(SET if Not eXist)

      - 특정 key 값이 존재하지 않을 경우 set
      - value가 없을 때만 값을 설정함
      - Lock을 획득하는 효과를 낼 수 있음

    - ##### 3.1.2 Pros & Cons

      - Pros
        - Timeout을 손쉽게 구현할 수 있음
        - Redis Client로 제공되서 별도의 설정 없이 간단히 구현 가능
      - Cons
        - Lock Timeout 설정이 되지 않음(SETNX)
        - Redis에 많은 부하가 발생(Spin Lock)


    - ##### 3.1.3 Code

      ```java
      @Component
      public class RedisLockRepository {
        private final RedisTemplate<String, String> redisTemplate;

        public RedisLockRepository(RedisTemplate<String, String> redisTemplate) {
            this.redisTemplate = redisTemplate;
        }

        public Boolean lock(Long key){
          return redisTemplate
                  .opsForValue()
                  .setIfAbsent(generated(key), "lock", Duration.ofMillis(3000));
        }

        public Boolean unlock(Long key){
            return redisTemplate.delete(generated(key));
        }

        private String generated(Long key) {
            return key.toString();
        }
      }
      ```

      ```java
      public void decrease(Long key, Long quantity) throws InterruptedException {
        while (!redisLockRepository.lock(key)) {
            Thread.sleep(100);
        }

        try {
            memberService.decrease(key, quantity);
        } finally {
            redisLockRepository.unlock(key);
        }
      }
      ```

  - #### 3.2 Redisson

    - pub/sub 방식
    - 재시도가 필요한 경우(실무)
    - 채널을 구독한 subscribe에게 Lock이 해제될 때마다 메세지를 전송하는 방식
    - 특정 Channel을 통해서 메세지를 바로 전달하기 때문에 보관하지 않음

    - ##### 3.2.1 Pros & Cons

      - Pros
        - Lock Timeout을 설정할 수 있음
        - pub-sub 방식으로 Lettuce에 비해 Redis에 부하가 덜 감
        - Library를 통해 분산락 기능을 제공함
      - Cons
        - 별도의 Dependency 추가 필요


    - ##### 3.2.2 Code

      ```java
      public void decrease(Long key, Long quantity) throws InterruptedException{
        RLock lock = redissonClient.getLock(key.toString());

        try {
          boolean available = lock.tryLock(20, 1, TimeUnit.SECONDS);

          if (!available) {
              System.out.println("lock 획득 실패!");
              return;
          }

          stockService.decrease(key, quantity);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        } finally {
            lock.unlock();
        }
      }
      ```


### 4. RDB Lock vs NoSQL Lock(Redis)
  - 성능
    - RDB << Redis
  - 비용
    - RDB >> Redis

### 마무리하면서
- 동시성 이슈를 해결하는 방법은 여러 가지가 있지만 각각의 장단점이 존재합니다. RDB, NoSQL Lock중에 성능과 비용 면으로 따지자면 서로의 장단점이 있겠지만 시스템의 비즈니스나 인프라 상황을 고려하여 우선적으로 고려하여 선택하는 것이 좋아보입니다. 다음 **Redis** 관련 포스트에서는 실제 토이프로젝트에서 구현한 코드와 성능 테스트를 통해 더 자세히 알아보겠습니다.

<br />

---

### Reference

- https://velog.io/@dlswns2480/Redis%EB%A5%BC-%ED%99%9C%EC%9A%A9%ED%95%9C-%EB%8F%99%EC%8B%9C%EC%84%B1-%EC%9D%B4%EC%8A%88-%EC%A0%9C%EC%96%B4-Lettuce-vs-Redisson
- https://velog.io/@nuyh99/%EC%8A%A4%ED%94%84%EB%A7%81%EC%97%90%EC%84%9C-%EB%8F%99%EC%8B%9C%EC%84%B1-%EB%AC%B8%EC%A0%9C%EB%A5%BC-%ED%95%B4%EA%B2%B0%ED%95%98%EB%8A%94-5%EA%B0%80%EC%A7%80-%EB%B0%A9%EB%B2%95
- https://velog.io/@fill0006/Spring-%EB%8F%99%EC%8B%9C%EC%84%B1-%EB%AC%B8%EC%A0%9C%EB%8D%B0%EC%9D%B4%ED%84%B0-%EC%A0%95%ED%95%A9%EC%84%B1-%EB%BF%8C%EC%85%94%EB%B3%B4%EA%B8%B0
- https://minwoo-it-factory.tistory.com/entry/Lock-%EC%A0%95%EB%A6%AC%EB%82%99%EA%B4%80%EC%A0%81-%EB%9D%BD%EA%B3%BC-%EB%B9%84%EA%B4%80%EC%A0%81-%EB%9D%BD-%EB%B6%84%EC%82%B0%EB%9D%BD-%EB%8D%B0%EB%93%9C%EB%9D%BD-%EB%B0%8F-%ED%99%9C%EC%9A%A9%EA%B9%8C%EC%A7%80
- https://minwoo-it-factory.tistory.com/entry/Lock-%EC%A0%95%EB%A6%AC%EB%82%99%EA%B4%80%EC%A0%81-%EB%9D%BD%EA%B3%BC-%EB%B9%84%EA%B4%80%EC%A0%81-%EB%9D%BD-%EB%B6%84%EC%82%B0%EB%9D%BD-%EB%8D%B0%EB%93%9C%EB%9D%BD-%EB%B0%8F-%ED%99%9C%EC%9A%A9%EA%B9%8C%EC%A7%80
- https://velog.io/@juhyeon1114/Spring-RedisRedisson-%EB%B6%84%EC%82%B0%EB%9D%BD%EC%9D%84-%ED%99%9C%EC%9A%A9%ED%95%98%EC%97%AC-%EB%8F%99%EC%8B%9C%EC%84%B1-%EB%AC%B8%EC%A0%9C-%ED%95%B4%EA%B2%B0%ED%95%98%EA%B8%B0
- https://ttl-blog.tistory.com/1581
- https://devhooney.tistory.com/111