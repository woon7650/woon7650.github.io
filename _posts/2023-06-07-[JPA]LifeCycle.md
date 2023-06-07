---
title:  "[JPA] EntityLifeCycle"
excerpt: "About Entity LifeCycle"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2023-06-07
---

### Entity Lifecycle이란 ?

<br />

![image info](/assets/img/jpaLifecycle.png)
<img src="/assets/img/jpaLifecycle.png" alt="" width="0" height="0">

> Entity 객체가 생성되어 영속성 컨텍스트(Persistence Context)에서 관리되는 과정


- Entity가 어떤 상태에 있는 지를 나타내며 상태 변화에 따라 Entity와 DB 간의 동기화 작업이 수행됨
- Entity 생명주기는 <mark style="background-color:#cccccc">Transient(비영속)</mark>, <mark style="background-color:#cccccc">Managed(영속)</mark>, <mark style="background-color:#cccccc">Detached(준영속)</mark>, <mark style="background-color:#cccccc">Removed(삭제)</mark> 4개의 상태로 이루어있음


<br />

---
### Entity Lifecycle 상태 종류

<br />

####*Transient(비영속)*

- Entity 객체가 생성되었지만 영속성 컨텍스트와 관련이 없는 상태
- 단순 객체 생성 상태로 JPA가 모르는 상태

```java
Example example = new Example();
```

<br />

####*Managed(영속)*

- Entity 객체가 영속성 컨텍스트에 저장되어 관리되고 있는 상태
- Entity Manager를 사용하여 영속성 컨텍스트에 등록
- 현재 상태에서는 Entity는 영속성 컨텍스트가 제공하는 Dirty Checking, 변경 추적, DB와 동기화 작업이 가능
- Transaction Commit시에 DB와 동기화 작업을 수행

```java
//Entity 영속화
em.persist(example);

//준영속 상태의 Enity를 다시 영속화
em.merge(example);
```

<br />

####*Detached(준영속)*

- 영속성 컨텍스트에 의해 관리되던 Entity가 영속성 컨텍스트에서 분리된 상태
- Entity는 영속성 컨텍스트가 제공하는 Dirty Checking, 변경 추적, DB와 동기화 작업이 불가능
- 영속성 컨텍스트와 관계가 없는 상태


```java
//영속화된 Entity 분리
em.detach(example);

//영속성 컨텍스트 닫기 및 초기화
em.close();
em.clear();
```

<br />


####*Removed(삭제)*

- 영속성 컨텍스트에 의해 관리되던 Entity가 DB에서 삭제된 예정인 상태
- Transaction Commit시 Entity가 DB에서 삭제

```java
//영속성 컨텍스트 및 DB에서 삭제
em.remove(example);
```

<br />

*<mark style="background-color:#cccccc">영속성 컨텍스트의 변경 내용은 em.flush()에 의해 DB에 반영됨</mark>*

---
### 영속성 컨텍스트(Persistence Context)에서 제공하는 기능 

<br />

- 1차 캐시
- DB와의 동일성 보장
- Transaction을 지원하는 쓰기 지연
- 변경 감지
- 지연 로딩

---
