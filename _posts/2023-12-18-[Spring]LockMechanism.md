---
title:  "Lock Mechanism"
excerpt: "[Spring] Lock Mechanism"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2023-12-18

---


#### 0. Lock Mechanism

> Locking is a mechanism in Java that allows a thread to exclusively acquire a lock on an object or a class, preventing other threads from accessing the locked object or class until the lock is released. In this blog, we will discuss the different types of locks in Java and provide examples to demonstrate how they can be used in a multi-threaded environment.

> Spring Conatiner는 Spring Framework의 핵심에 있다. Spring Container는 객체를 만들고, 연결하고, 구성하고, 만들어지는 것부터 파괴될 때까지 완전한 수명 주기를 관리할 것이다. Spring Container는 DI를 사용하여 응용 프로그램을 구성하는 구성 요소를 관리한다. 이 객체들을 Spring Beans라고 부른다.


- Locking : 동시에 여러 개의 Transaction이 접근했을 때 데이터의 일관성이 손상되는 것을 막기 위한 수단


<br />

---


### 1. Optimistic Lock

> 기본적으로 데이터 갱신 시 충돌이 발생하지 않을 것으로 간주하는 Lock 방식


- 트랜잭션(Transaction) 충돌이 발생하지 않을 것이라고 낙관적으로 생각하는 데이터 제어 방법
- 트랜잭션(Transaction) 충돌을 감지하면 Exception을 발생시켜서 트랜잭션(Transaction)을 중지시킴
- DB에서 제공해주는 특징을 이용하는 것이 아닌 Application Level에서 잡아주는 Lock(다른 트랜잭션(Transaction)의 접근이 가능)
- 데이터에 대하여 Lock을 선점한다(X)보다는 트랜잭션(Transaction) 충돌을 감지한다 (O) 
- 트랜잭션(Transaction) 충돌의 감지는 조회한 데이터의 버전 값을 통해 이루어짐


<br />

##### 1.1 How Optimistic Lock Works

- 트랜잭션(Transaction) 충돌에 대한 감지는 조회한 데이터의 버전 값을 통해 이루어짐
- 업데이트(Update)시에 현재 버전을 확인하여 충돌을 감지하고 현재 버전 값을 증가시켜 데이터 변경여부를 남김
- 발생할 수 있는 예외(Exception) : OptimisticLockException

![image info](/assets/img/optimistic.png)
<img src="/assets/img/optimistic.png" alt="" width="0" height="0">

1. Alice와 Bob이 같은 데이터를 읽어 버전 1을 가져감
2. Bob이 먼저 데이터에 대한 update를 실행하면서 버전 2로 올라감
3. Alice도 update를 실행하면서 버전 2로 올림
4. 버전 2가 존재하기 때문에 Optimistic Lock Exception이 발생

<br />

---


### 2. Pessimistic Lock

> 데이터 갱신 시 트랙잭션(Transaction) 충돌이 발생할 것을 예상하고 데이터에 대한 Lock을 미리 선점하는 Lock 방식

- 트랙잭션(Transaction) 충돌을 예상하고 데이터에 대한 Lock을 미리 선점하는 Lock 방식
- 트랙잭션(Transaction) 시작시 Shared Lock 또는 Exclusive Lock을 걸고 시작하는 Lock 방식
- Lock이 걸려있는 상태에서는 다른 사용자의 update 불가능
- 데이터에 대한 트랙잭션(Transaction)이 완전히 해제(commit)이 되었을 때 변경이 가능


<br />

##### 2.1 How Pessimistic Lock Works

- 데이터 조회 시 <mark style="background-color:#cccccc">SELECT FOR UPDATE WAIT</mark> 쿼리를 수행하여 데이터 Lock을 점유
 - 다른 트랙잭션(Transaction)에 의해 Lock이 걸린다면 지정한 시간만큼 대기
 - 지정한 시간 초과시 Exception 발생
- Lock을 선점한 트랙잭션(Transaction)이 commit 되는 시점까지 대기
- Lock의 종류
  - Exclusive Lock : 다른 사용자의 읽기, 수정, 삭제 Lock
  - Shared Lock : 다른 사용자가 동시에 읽을 수는 있지만 수정, 삭제 Lock

![image info](/assets/img/pessimistic.png)
<img src="/assets/img/pessimistic.png" alt="" width="0" height="0">

1. Alice가 먼저 데이터를 읽음
2. Bob이 Alice가 읽은 데이터에 대하여 읽음
3. Bob이 해당 데이터에 대하여 Update 요청
4. 하지만 해당 데이터에 대하여 Alice쪽에서 이미 shared lOCK을 잡고 있기 때문에 Block
5. Alice측에서 트랙잭션(Transaction) 해제(commit)
6. Blocking이 됬던 Bob의 Update 요청이 정상적으로 처리

<br />

---

### 3. Optimistic Lock vs Pessimistic Lock

- 기준 : 해당 데이터에 대한 읽기와 쓰기 비율
  - Optimistic Lock : 데이터의 읽기의 비율이 높을 때 사용
  - Pessimistic Lock : 데이터의 쓰기의 비율이 높을 때 사용


### Refernce 
- https://velog.io/@lsb156/JPA-Optimistic-Lock-Pessimistic-Lock
- https://sabarada.tistory.com/175
- https://junhyunny.github.io/information/lock-mechanism/
- https://sabarada.tistory.com/175
- https://backtony.github.io/interview/2021-11-23-interview-8/