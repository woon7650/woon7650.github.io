---
title: "Isolation Level and Propagation of Transaction"
excerpt: "[Spring] 트랜잭션의 격리수준과 전파수준"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2024-08-21
---

##### RELATED POST
-  [Transaction](/blog/Database-transaction/)
-  [@Transactional](blog/Spring-@Transactional_/)


#### 0. 들어가면서

  - 이직 준비를 하면서 면접에서 자주 등장하는 Backend CS 관련 지식을 정리하고 자세히 알아보고자 합니다.
  - 지난 시간 동시성 이슈 해결 방법 중 synchronzied를 다루면서 JPA @Transactional의 격리수준과 전파수준에 대해 정리하고자 합니다.

  - ##### @Transactional Options

    - Isolation Level
    - Propagation
    - Timeout
    - Read-only
    - rollback condition

  - ##### Concurrency Issue

    - Dirty Read
      - 변경사항이 반영되지 않은 값은 다른 Transaction에서 읽도록 허용할 경우 발생(데이터 불일치)
    - Non-Repeatable Read
      - 한 Transaction 내에서 값을 재조회할 때 같은 쿼리가 다른 값을 반환함
      - Transaction이 끝나기 전에 수정사항이 commit되어 Transaction내에서 쿼리 결과가 일관성이 없음
    - Phantom Read
      - 다른 Transaction에서의 Insert, Delete 작업이 진행된 경우 동일하나 쿼리에서 다른 행을 반환함

<br />

---

#### 1. Isolation Level

  - 여러 Transaction이 동시에 실행될 때 발생할 수 있는 동시성 issue들을 제어하는 수준
  - Transaction의 ACID 원칙 중 Isolation을 지원하기 위해 제공
    
  - ##### 1.1 Conception

    - Isolation Level High
      - 높은 직렬화
      - 높은 안정성
      - 동시 처리 가능 Transaction 수 감소
      
    - Isolation Level Low
      - 낮은 직렬화
      - 낮은 안정성
      - 동시 처리 가능 Transaction 수 증가

  - ##### 1.2 Category

    - ###### Default
      - DBMS의 Isolation Level을 따름

    - ###### READ_UNCOMMITED
      - 가장 낮은 수준의 Isolation level
      - commit 되지 않은 데이터에 대한 Read 허용
      - 동시성 issue 모두 발생(Dirty Read, Non-Repeatable Read, Phantom Read)
      - 
        ```java
        @Transactional(isolation = Isolation.READ_UNCOMMITED)
        ```

    - ###### READ_COMMITED
      - commit된 데이터만 Read 허용
      - 동시성 issue 발생(Non-Repeatable Read, Phantom Read)
      - Default : MS SQL, Postgres, Oracle, SQL Server
      - 
        ```java
        @Transactional(isolation = Isolation.READ_COMMITED)
        ```

    - ###### REPEATABLE_READ
      - Transaction이 완료될 때까지 SELECT문이 사용된 모든 데이터 Shared Lock 처리
      - 동시성 issue 발생(Phantom Read)
      - 조회하는 데이터 정합성
      - Default : MySQL
      - 
        ```java
        @Transactional(isolation = Isolation.REPEATABLE_READ)
        ```

    - ###### SERIALIZABLE
      - 최고 높은 수준의 Isolation Level
      - 동시성 issue 모두 방지(Dirty Read, Non-Repeatable Read, Phantom Read)
      - 성능 저하 우려
      - 가장 낮은 동시 처리량
      - 
        ```java
        @Transactional(isolation = Isolation.SERIALIZABLE)
        ```


  - ##### 1.2 Snapshot Isolation

    - 비표준 Isolation Level
    - MVCC의 한 종류로 변경되는 데이터의 Multi-version 관리
    
    - ##### 1.2.1 Process
      - Transaction의 시작 시점을 기준으로 Snapshot 생성
      - Transaction 진행 도중 변경된 데이터들을 DB에 즉시 반영하지 않고 Snapshot으로 만들어두었다가 commit시에 DB에 반영
      - Write Conflict 발생 경우(First-Committer Win)
        - 다수의 Transaction이 동일한 데이터에 대해 write 작업을 진행된 경우
        - 먼저 commit된 Transaction만이 DB에 반영되고 이후의 Transaction은 Rollback됨

  - ##### 1.3 Trade-off

    ![image info](/assets/img/isolationLevel.png)
    <img src="/assets/img/isolationLevel.png" alt="" width="0" height="0">

<br />

---

#### 2. Propagation

  - Transaction을 시작하고 멈추는 실행 범위를 지정

  - ##### 2.1 TransactionManager

    - getTransaction을 통해 propagation 설정 내용을 기반으로 Transaction을 가져옴

  - ##### 2.2 Category
  
    - ###### REQUIRED(Default)
      - 항상 Transaction이 실행
      - 기존 Transaction 존재 o : 해당 Transaction을 사용
      - 기존 Transaction 존재 x : 새로운 Transaction을 생성
        ```java
        @Transactional(propagation = Propagation.REQUIRED)
        ```

    - ###### SUPPORTS
      - 기존 Transaction 존재 o : 해당 Transaction을 사용
      - 기존 Transaction 존재 x : Transaction 없이 실행
        ```java
        @Transactional(propagation = Propagation.SUPPORTS)
        ```

    - ###### NOT_SUPPORTED
      - 항상 Transaction이 없이 실행
      - 기존 Transaction 존재 o : 현재 Transaction 종료까지 일시중단(보류)
        ```java
        @Transactional(propagation = Propagation.NOT_SUPPORTED)
        ```

    - ###### REQUIRES_NEW
      - 항상 새로운 Transaction이 실행
      - 기존 Transaction 존재 o : 현재 Transaction 종료까지 일시중단(보류)
      - 기존 Transaction 존재 x : 새로운 Transaction 생성
        ```java
        @Transactional(propagation = Propagation.REQUIRES_NEW)
        ```

    - ###### MANDATORY
      - 항상 Transaction이 실행
      - 기존 Transaction 존재 o : 해당 Transaction을 사용
      - 기존 Transaction 존재 x : Exception을 발생
      - 독립적으로 Transaction을 진행하면 안되는 경우 사용
        ```java
        @Transactional(propagation = Propagation.MANDATORY)
        ```

    - ###### NEVER
      - 항상 Transaction이 없이 실행
      - 기존 Transaction도 존재하면 안되고 있으면 Exception을 발생
        ```java
        @Transactional(propagation = Propagation.NEVER)
        ```

    - ###### NESTED
      - 항상 Transaction이 실행
      - 기존 Transaction 존재 o : 해당 Transaction에 SAVE POINT를 생성
      - 기존 Transaction 존재 x : 새로운 Transaction을 생성
      - 기존 Transaction 내에 중첩 Transaction 생성(Transaction 안에 Transaction 생성)
      - 비즈니스 로직에서 Exception 발생 시 SAVE POINT로 Rollback
      - 하위 Transaction은 상위 Transaction에 영향을 받지만, 상위 Transaction은 하위 Transaction에게 영향을 덜 받도록 구축하는 형태
      - 기존 트랜잭션이 없는 경우, REQUIRED 처럼 새로운 트랜잭션을 생성

      - DB 및 JDBC Driver에서 지원하는지 여부 확인이 필요
        ```java
        @Transactional(propagation = Propagation.NESTED)
        ```

<br />

---



### Reference


- https://jindory.tistory.com/entry/Spring-Transaction-Propagation-Model%EA%B3%BC-Isolation-Level
- https://devocean.sk.com/blog/techBoardDetail.do?ID=163799
- https://ryanwoo.tistory.com/44
- https://ojava.tistory.com/207
- https://km-so-yeon.github.io/posts/DB-Propagation-Isolation/