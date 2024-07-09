---
title: "Redis"
excerpt: "[Spring] Redis "

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2024-07-09
---

#### 0. 들어가면서

> Redis is an advanced key-value store. It is similar to memcached but the dataset is not volatile, and values can be strings, exactly like in memcached, but also lists, sets, and ordered sets. All this data types can be manipulated with atomic operations to push/pop elements, add/remove elements, perform server side union, intersection, difference between sets, and so forth. Redis supports different kind of sorting abilities.

> Redis는 고급 키-값 저장소입니다. Memcached와 비슷하지만 데이터 세트는 휘발성이 아니며 값은 Memcached와 정확히 같은 문자열일 수 있지만 목록, 세트 및 정렬된 세트도 가능합니다. 이 모든 데이터 유형은 원자적 연산으로 조작하여 요소를 푸시/팝하고, 요소를 추가/제거하고, 서버 측 합집합, 교집합, 세트 간 차이 등을 수행할 수 있습니다. Redis는 다양한 종류의 정렬 기능을 지원합니다.

- 웹 어플리케이션 성능 향상에 많이 기여하는 캐싱의 종류에 대해 자세히 알아보고자 함
- 이번 글에서는 Redis와 Memcahced 중에 Redis에 대해 먼저 다뤄보고자 함 

<br />

---

### 1. Redis(Remote Dictionary Server)


- #### 1.1 Conception

  - Dictionary(Key-Value) 형태의 영속성을 지원하는 비관계형 데이터베이스 관리 시스템(NoSQL DBMS)
  - 인-메모리(In-memory) 데이터 구조 저장소
  - Single Thread이기 때문에 시간이 오래걸리는 연산을 지양(O(N) 연산 : Keys, Flush, GetAll)

- #### 1.2 Features

  - #### 1.2.1 Inmemory Database
    - Redis는 Memory상에 저장하기 때문에 빠른 응답 속도를 제공함
    - 평균 읽기 및 쓰기 작업 속도가 1ms

  - #### 1.2.2 Variable Date Type
    - String, List, Hash, Set, Bitmap, JSON 등.. 다양한 데이터 구조를 지원함
    - 다양한 데이터 처리 요구사항을 처리하기 가능
    
  - #### 1.2.3 Persistence
    - 디스크에 Data를 주기적으로 저장하여 지속성을 제공함
    - 시스템 장애 시에도 Data 손실을 방지할 수 있음

    - #### 1.2.3.1 Problem
      - Redis는 Memory 내에 데이터를 저장 -> 시스템 종료, 재시작, 장애 발생 시 데이터가 영구적으로 보존되지 않음

    - #### 1.2.3.2 Solution
      - RDB(Redis DataBase) SNAPSHOT
        - 주기적으로 RDB의 SNAPSHOT을 Disk에 저장하는 방법
        - 필요 시 Data를 복구하는데 사용
      - AOF(Append Only File) LOG
        - 모든 변경 사항을 로그 파일에 기록하는 방법
        - Redis 서버 재가동 시 LOG 실행하여 Data 복구

    - #### 1.2.3.3 Summary
      - RDB SNAPSHOT과 AOF LOG 함께 사용할 수 없음
      - 모두 사용하지 않거나 두가지 옵션을 각각 또는 모두 사용 가능
      - 비즈니스 환경에 맞춰 조절하여 사용함
        
  - #### 1.2.4 Pub/Sub Messaging
    - Publish-Subscribe을 지원하여 메세지 브로커로 사용할 수 있음
    - 이벤트 기반 시스템을 구축하거나 메세지 전달을 활용 가능
        
  - #### 1.2.5 Single Thread
    - 한 번에하나의 명령어만 처리함
    - Race Condition이 거의 발생하지 않음
    - 시간 복잡도가 O(n)인 연산에 대하여 주의해야 함
    - Side Effect가 거의 없는 매우 안정적인 서비스 구축 가능
        
  - #### 1.2.6 Clustering

    - 데이터 샤딩과 레플리케이션을 통해 고가용성 및 확장성을 제공하는 클러스터 구성 가능
        
  - #### 1.2.7 LRU Caching & Expiration

    - 데이터를 자동으로 관리하기 위해 LRU(Least Recently Used) 알고리즘과 만료시간(Time-to-Live)을 지원

- #### 1.3 Usage

  - Caching : JWT, Session.. 일정 주기로 갱신해도 되는 데이터, 동일한 연산에 따른 결과
  - Real Time Analyze : 순위, 실시간 이벤트 로그 처리, 방문자 수 계산
  - Pub/Sub : 실시간 채팅, 이벤트 메세징 처리
  - Queue : 우선 순위 큐, 이메일 전송

<br />

---
### 2. Redis Configuration in Spring

- #### 2.1 Dependency

  - maven(pom.xml)
    ```java
    <dependency>    
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
    ```

  - gradle(build.gradle)
    ```java
    implementation 'org.springframework.boot:spring-boot-starter-data-redis'
    ```

- #### 2.2 Global Variable

  - .properties
    ```java
    spring.redis.host = 127.0.0.1
    spring.redis.port = 6379
    spring.redis.timeout = 1
    ```

  or

  - .yml
    ```java
    spring :
      redis :
        host : 127.0.0.1
        port : 6379
        timeout : 1
    ```

- #### 2.3 RedisConfig

  ```java
  @Configuration
  @EnableCaching
  public class RedisConfig{

    @Value("${spring.redis.host}")
    private String host;

    @Value("${spring.redis.port}")
    private int port;

    @Value("${spring.redis.timeout}")
      private int timeout;

    @Bean
    //Lettuce와 Jedis 중에 설정
    public RedisConnectionFactory redisConnectionFactory(){
      return new LettuceConnectionFactory(host,port)
    }

    @Bean
    public RedisTemplate<String, Object> redisTemplate() {
      RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
      redisTemplate.setConnectionFactory(redisConnectionFactory());

      // key,value경우의 Serializer
      redisTemplate.setKeySerializer(new StringRedisSerializer());
      redisTemplate.setValueSerializer(new StringRedisSerializer());

      // Hash경우의 Serializer
      redisTemplate.setHashKeySerializer(new StringRedisSerializer());
      redisTemplate.setHashValueSerializer(new StringRedisSerializer());

      // 모든 경우
      redisTemplate.setDefaultSerializer(new StringRedisSerializer());

      return redisTemplate;
    }

    @Bean
    public RedisCacheManager redisCacheManager(RedisConnectionFactory redisConnectionFactory) {
      RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig()
      .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSera)
    }

    @Bean(name = "cacheManager")
    public RedisCacheManager cacheManager(RedisConnectionFactory connectionFactory) {
        RedisCacheConfiguration configuration = RedisCacheConfiguration.defaultCacheConfig()
        //value가 null이면 Cache 안함
        .disableCachingNullValues() 
        //Cache Default Expiration 설정
        .entryTtl(Duration.ofHours(timeout)) 

        //Redis Cache Key 저장방식을 StringSeriallizer로 지정
        .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer())) 
        //Redis Cache Value 저장방식을 GenericJackson2JsonRedisSerializer로 지정
        .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer())); 


        return RedisCacheManager.RedisCacheManagerBuilder.fromConnectionFactory(connectionFactory).cacheDefaults(configuration).build();
    }
  }
  ```

  - Spring Boot 2.0이상 부터는 auto-configuration으로 관련된 Bean(redisConnectionFactory, RedisTemplate)이 자동 생성됨
  - @EnableCaching: Application Caching 기능을 켬
  


- #### 2.4 RedisConnectionFactory

  - Redis DB와 연결을 위해 Redis Client인 Lettuce와 Jedis 사용
  - 비동기 동작 및 성능의 이유로 Lettuce 드라이버 사용 권장

- #### 2.5 Spring Data Redis

  - Spring Data Redis는 Redis의 데이터에 접근할 수 있도록 RedisTemplate, RedisRepository를 제공함

  - #### 2.5.1 RedisTemplate

    - Redis의 모든 명령을 실행 가능
    - Operations을 제공해 Redis의 데이터에 접근
    - redis에 데이터 저장시 사용
    - 자료구조별 Operations를 제공
      - opsForValue : String 제공 클래스
      - opsForList : List 제공 클래스
      - opsForSet : Set 제공 클래스
      - opsForZSet : Sorted Set 제공 클래스
      - opsForHash : Hash 제공 클래스
  
  - #### 2.5.1 RedisRepository

    - Spring Data JPA와 같이 미리 정의된 메소드를 통해 Redis에 접근 가능
      - findById
      - save
    - extends
      - CRUDRepository


- #### 2.6 CacheManager

  - Cache에서 Redis를 사용하기 위해 설정
  - RedisCacheManager를 Bean으로 등록하면 기본 CacheManager를 RedisCacheManager로 사용함
  
  - ##### 2.6.1 Dependency

    ```java
    implementation 'org.springframework.boot:spring-boot-starter-cache'
    ```

- #### 2.7 Usage
  
  ```java
  @Service
  public class RedisUtils {

    @Autowired
    private RedisTemplate<String, Object> redisTemplate;

    public void setData(String key, String value, Long expiredTime){
        redisTemplate.opsForValue().set(key, value, expiredTime, TimeUnit.DAYS);
    }

    public String getData(String key){
        return (String) redisTemplate.opsForValue().get(key);
    }

    public void deleteData(String key){
        redisTemplate.delete(key);
      }
  }
  ```

  ```java
  redisUtils.setData(..)
  redisUtils.getData(..)
  redisUtils.deleteData(..)
  ```

<br />

---

### Refernce

- https://mungto.tistory.com/460
- https://ssoco.tistory.com/19
- https://velog.io/@dev_lee/Redis-%EB%A0%88%EB%94%94%EC%8A%A4-%EC%86%8C%EA%B0%9C%EC%99%80-%ED%8A%B9%EC%A7%95-%EB%B0%8F-%EC%9E%A5%EC%A0%90-%EA%B7%B8%EB%A6%AC%EA%B3%A0-%EC%8B%A4%EC%A0%9C-%ED%99%9C%EC%9A%A9-%EC%82%AC%EB%A1%80
- https://ittrue.tistory.com/317
- https://velog.io/@haebing0309/Spring-Boot%EC%97%90%EC%84%9C-Redis-%EC%84%A4%EC%A0%95
- https://bepoz-study-diary.tistory.com/418
- https://programmingiraffe.tistory.com/171