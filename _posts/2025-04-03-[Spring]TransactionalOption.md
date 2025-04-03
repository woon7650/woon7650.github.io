---
title: "Performance Optimization with ReadOnly Transaction"
excerpt: "[Spring] Transacional Options and Performance in master-slave structure"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2025-04-03
---

##### RELATED POST
- [Transaction isolation & propagation](blog/Spring-isolationLevel/)
- [@Transactional](blog/Spring-@Transactional_/)

<br />

---

### 들어가면서
  - 최근 프로젝트에서 외부에서 사용 가능한 REST API를 설계하면서 데이터의 일관성과 여러 Exception 처리를 위해 @Transactional의 Option들을 적절히 조합하여 사용했습니다. 또한 제공되는 Option들을 알맞은 상황에 적절하게 조합하여 사용할 수 있도록 정리하고 연관된 포스트 작성을 통해 차례대로 정리해보고자 합니다.
  - 추후 포스트에서는 @Transactional(AOP)를 사용하면서 주의할 점과 사용하면서 있어서 마주했던 문제와 해결 방법에 대해서도 다루고자 합니다.

<br />

---
> ### @Transactional Options

  - #### Propagation & Isolation
    - Related Post 참고

  - #### rollbackFor

    - Transaction은 Unchecked Exception과 Checked Exception에 대하여 다르게 동작한다 
      - Unchecked Exception(RuntimeException, Error) 발생할 경우 rollback
      - Checked Exception 발생할 경우 rollback 하지 않고 commit
    - Checked Exception(CustomException이나 Exception)에 대하여 rollback을 지정할 수 있다
      ```java
      @Transactional(rollbackFor = {CustomException.class, Exception.class})
      ```
    - CustomException 및 Exception과 하위 Exception 발생할 경우 rollback


  - #### noRollbackFor
  
    - rollbackFor의 반대 개념으로 rollback하면 안되는 예외 설정
    - 설정한 예외의 하위 예외들도 대상에 포함된다
      ```java
      @Transactional(noRollbackFor = RuntimeException.class)
      ```
    - RuntimeException 발생 및 하위 예외 발생할 경우 rollback을 막는다

  - #### Timeout

    - Transaction 수행 시간에 대하여 Timeout을 지정할 수 있다
    - Timeout을 지정한 시간 내에 해당 메소드 수행을 완료하지 못할 경우 rollback
      ```java
      @Transactional(timeout=60) //Default : -1(no timeout)
      ```

  - #### readOnly

    - Transaction은 읽기 쓰기가 모두 가능 생성
      ```java
      @Transactional(readOnly = true) //Defulat : false
      ```
    - Optimization
      - Entity의 변경, 삭제를 방지하고 성능 최적화가 가능
      - 불필요한 Lock이나 리소스 소모를 방지하고 읽기 성능을 향상
 
<br />

---
> ### Performance Optimization with readOnly in Master-Slave Structure

  - ### Transaction Efficiency
    
    - 쓰기 작업 차단
      - JdbcTemplate throw Exception : 읽기 전용 Transaction 내에 쓰기 작업 발생할 경우 예외 발생
      - 의도치 않은 데이터 변경을 방지하고 Transaction 관리의 일관성 보장
    - no Flush : Hibernate는 변경에 사용되는 Flush를 Commit 시점에 호출하지 않는다
      - 불필요한 Database I/O 감소
    - no SnapShot : 변경 감지를 위한 SnapShot 객체을 생성하지 않는다
      - Memory 사용량을 줄이고 불필요한 리소스 소모 방지

  - #### Load Balancing
    - Master Database : 쓰기 작업(create, update, delete) 처리
    - Slave Database : Master에서 발생한 변경 사항을 복제하여 읽기 작업 처리

    - readOnly Transaction
      - Slave Database는 읽기 작업을 처리해 Master Database의 부하를 줄인다(읽기 전용 Transaction을 분배함으로써 성능을 높임)
      - Master Database는 쓰기 작업에 집중하며 마스터 서버의 자원을 최적화한다

  - #### Minimizing Transaction Locking
    - No Data Modification : 데이터 변경이 없어 Transaction 간 락 경합(Lock Contention)이 발생 방지
    - Optimistic Lock : 데이터 변경이 없기 때문에 낙관적 락을 활용하여 동시성 문제를 해결하고 성능을 최적화

  - #### Horizontal Scaling

    - 트래픽이 급증할 경우 읽기 작업을 처리하는 서버를 추가하여 시스템이 더 많은 요청을 동시에 처리할 수 있도록 만든다
    - Slave Server 추가 : Slave Server를 여러 대 추가하면 읽기 요청을 여러 서버가 나누어 처리하게 되어 서버 하나에 걸리는 부하가 줄어든다(읽기 요청이 많은 시스템에 적합)
    - Read/Write Splitting : 읽기 작업, 쓰기 작업을 각각 Slave, Master Server로 보내는 방식을 통해 처리 능력이 향상

  - #### Connection Pooling

    - Connection Pool : Database와 Application 간의 연결을 관리하는 방법
    - 읽기 전용 Transaction을 통해 새로운 연결 없이 기존 연결을 재사용해 연결 획득 시간을 최소화

  - #### Cache Optimization

    - 읽기 전용 Transaction의 Cache 사용은 데이터의 변경이 없기 때문에 Database에 불필요한 접근(디스크 I/O)을 줄이고 보다 더 빠르게 데이터를 불러올 수 있다

  - #### Replication Lag Management

    - 복제 지연(Replcation Lag) : Slave가 Master의 데이터를 받는 동안 지연되는 시간
      - Replication Lag가 너무 크면 데이터의 정확성이 떨어질 수 있다
    - 읽기 전용 Transaction은 최신 데이터를 항상 필요로 하지 않기 때문에 Replication Lag를 일정 수준까지 허용 가능

<br />

---
