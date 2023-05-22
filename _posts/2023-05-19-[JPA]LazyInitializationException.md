---
title:  "[JPA] LazyInitializationException"
excerpt: "About LazyInitializationException"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2023-05-18
---

## LazyInitializationException이란 ?

<br />

- JPA에서 <mark style="background-color:#cccccc">Lazy Loading</mark>된 Entity에 접근할 때 발생할 수 있는 Exception
- <mark style="background-color:#cccccc">FetchType.LAZY</mark>를 사용하여 Entity를 지연 로딩할 때 발생할 수 있는 Exception

---
## 예외 발생 경우

#### 1. 영속성 컨텍스트가 닫혀있는 경우
- Lazy Loading된 Entity에 접근하려는 시점에 영속성 컨텍스트가 닫혀있는 경우, 즉시 로딩이 아닌 경우 <mark style="background-color:#cccccc">LazyInitializationException</mark>이 발생할 수 있습니다.


#### 2. Entity가 Proxy로 초기화되지 않은 경우
-  Lazy Loading된 Entity가 영속성 컨텍스트에 존재하지만 아직 로딩되지 않은 상태에서 접근하려고 할 때 <mark style="background-color:#cccccc">LazyInitializationException</mark>이 발생할 수 있습니다. 이는 영속성 컨텍스트가 해당 Entity를 <mark style="background-color:#cccccc">Proxy</mark>로 감싸서 Lazy Loading을 지원하기 때문입니다.

<br />

---

## Solution


#### 1. Transaction
- Transaction 범위 내에서 연관된 Entity에 접근하도록 유지함
- @Transactional annotation을 사용하여 트랜잭션을 설정하거나, 트랜잭션을 직접 관리하는 방식을 사용할 수 있음

#### 2. Initialize
- Lazy Loading Entity를 초기화합니다.
- 영속성 컨텍스트가 열려있는 상태에서 연관된 Entity에 접근하여 Database에서 로딩하는 방식
- Transaction을 시작하고, 접근한 Entity에 대한 적절한 method를 호출하여 초기화할 수 있습니다.

#### 3. Fetch Join
- Fetch Join을 사용하여 Lazy Loading된 연관 Entity를 함께 불러옴


---
