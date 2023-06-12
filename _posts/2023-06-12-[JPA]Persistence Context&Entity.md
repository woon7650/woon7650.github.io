---
title:  "[JPA] Persistence Context & Entity"
excerpt: "About Persistence Context & Entity"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2023-06-12
---

### 영속성 컨텍스트(Persistence Context)

>- 영속성 컨텍스트(Persistence Context)는 Entity를 관리하는 영역이다.
>- JPA에서 Entity의 영속성을 관리하기 위한 기능을 제공
>- EntityManager를 통해 Entity를 영속성 컨텍스트(Persistence Context)에서 보관, 관리

---

#### 영속성 컨텍스트(Persistence Context)가 제공하는 기능

<br/>

*1차 캐시*

- 영속성 컨텍스트(Persistence Context)는 1차 캐시를 제공
- DB에서 조회한 Entity를 1차 캐시에 저장
- Entity를 조회 시 1차 캐시에 존재한다면 DB까지 접근하지 않고 1차 캐시에서 데이터를 가져옴

<br />

*지연 로딩(Lazy Loading)*

- 영속성 컨텍스트(Persistence Context)는 지연 로딩을 지원
- 연관된 Entity를 실제로 사용하기 전까지 로딩을 지연하고 실제 사용하는 시점에 로딩해옴

<br />


*변경 감지(Dirty Checking)*

- 영속성 컨텍스트(Persistence Context)는 Entity의 변경을 감지하여 DB에 자동으로 반영하는 기능을 지원
- DB에 반영되는 시점은 Transaction을 Commit하는 시점에 Entity의 변경사항을 DB에 자동으로 동기화

<br />

*트랜잭션 쓰기 지연*

- 영속성 컨텍스트(Persistence Context)는 Transaction 쓰기 지연을 지원
- Entity가 변경되는 시점이 아닌 Transaction Commit 시점까지 지연했다가 DB에 반영
  
<br />

*플러시(Flush)*

- 영속성 컨텍스트(Persistence Context)는 DB에 동기화하는 Flush 기능 지원
- Flush는 Transaction Commit 전에 자동으로 수행되며 Entity 변경사항을 DB에 적용

<br />

*쓰기 지연*

- 영속성 컨텍스트(Persistence Context)는 Entity를 DB에 동기화하는 작업을 지연하는 기능 지원
- 변경사항이 있는 Entity를 지연 버퍼에 저장하고 원하는 시점에 한 번에 DB에 동기화

---

#### Entity 등록, 조회, 수정, 삭제 과정

<br />

![image info](/assets/img/jpaLifecycle.png)
<img src="/assets/img/jpaLifecycle.png" alt="" width="0" height="0">

<br />

*Entity 등록*

1. <mark style="background-color:#cccccc">persist()</mark>를 통해서 Entity 영속화
2. 1차 캐시에 @Id와 Entity를 저장하고 <mark style="background-color:#cccccc">쓰기 지연 SQL 저장소</mark>에 INSERT SQL문 저장
3. Transaction Commit
4. EntityManager는 영속성 컨텍스트를 <mark style="background-color:#cccccc">flush()</mark>
5. 영속성 컨텍스트의 변경 내용을 DB에 동기화


<br />

*Entity 조회*

1. <mark style="background-color:#cccccc">find()</mark>를 통해서 Entity 조회
2. 1차 캐시에 @Id를 통해서 Entity 조회
3. 1차 캐시에 존재한다면 그대로 반환, 1차 캐시에 존재하지 않는다면 DB에서 조회
4. DB에서 조회한 data로 Entity를 생성하여 1차 캐시에 저장
5. 조회한 Entity 반환


<br />


*Entity 수정*

1. Transaction Commit 후에 EntityManager 내부에서 <mark style="background-color:#cccccc">flush()</mark> 호출
2. Entity와 SnapShot을 비교하여 변경된 Entity를 조회
3. 변경된 Entity는 수정 쿼리를 생성하여 <mark style="background-color:#cccccc">쓰기 지연 SQL 저장소</mark>에 보냄
4. <mark style="background-color:#cccccc">쓰기 지연 SQL 저장소</mark>의 SQL을 DB에 보냄
5. Transaction Commit

<br />

*Entity 삭제*

1. <mark style="background-color:#cccccc">find()</mark>를 통해서 해당 Entity 조회
2. <mark style="background-color:#cccccc">remove()</mark>를 통해서 조회한 Entity 삭제

---

#### 영속성 컨텍스트(Persistence Context) 제공하는 메소드

<br />


#### Flush

- flush: 영속성 컨텍스트의 변경 내용을 DB와 동기화하는 작업

*영속성 컨텍스트를 Flush하는 방법*
1. flush로 강제 호출
2. Transaction Commit시 flush 자동 호출(JPA)
3. JPQL 쿼리 실행시 flush 자동 호출


#### Detach

- detach: 해당 Entity를 준영속 상태로 전환(1차 캐시부터 쓰기 지연 SQL 저장소까지 Entity 관련 정보 제거)


#### Clear

- clear : 영속성 컨텍스트를 전부 초기화(영속성 컨텍스트의 모든 Entity를 준영속 상태로 전환)


#### Close

- close : 영속성 컨텍스트를 종료

<br />

<mark style="background-color:#cccccc">준영속 상태에서 영속성 컨텍스트가 지원하는 기능을 사용할 수 없지만 식별자 값은 가지고 있음</mark>

---
