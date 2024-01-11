---
title:  "Deadlock"
excerpt: "[Spring] Deadlock"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2024-01-11

---


#### 0. 들어가면서

NFT Market Project를 진행하면서 교착 상태(Deadlock) 관련된 문제 때문에 어려움을 겪었습니다. Blockchain network상의 nft들의 상태를 스케쥴러(Scheduler)로 동기화(sync)를 지속적으로 진행함에 있어서 사용자가 사용하는 지갑이 소유한 nft의 개수가 5000개를 넘어가기 시작하면서 트랜잭션(Transaction) 진행시간이 길어졌습니다. 트랜잭션(Transaction)이 길어지면서 nft에 접근이 필요한 다른 서비스들로 장애가 영향을 주는 상황이 발생했습니다. 이번 기회에 교착 상태(Deadlock)에 대하여 정리하고자 합니다.


---

### 1. Deadlock?

> 두 개 이상의 프로세스(Process)나 스레드(Thread)가 서로 자원을 얻지 못해서 다음 처리를 하지 못하는 상태를 뜻함. 무한히 다음 자원을 기다리게 되는 상태를 말한다. 시스템적으로 한정된 자원을 여러 곳에서 사용하려고 할 때 발생한다.

![image info](/assets/img/deadlock.png)
<img src="/assets/img/deadlock.png" alt="" width="0" height="0">

- Process1과 Process2가 Resource1,2를 모두 얻어야 할 경우 Deadlock 발생
- Case Of Process1 : Resource1을 얻음, Resource2를 기다림
- Case Of Process2 : Resource1을 기다림, Resource2를 얻음
- 각각의 Process가 필요로 하는 Resource가 다른 Process에 할당이 되어 있어서 무한정 대기상태(wait)에 빠짐 -> 교착 상태(Deadlock) 발생

<br />

---


### 2. Conditions for Deadlock

- 교착 상태(Deadlock)는 한 시스템 내에서  4가지 조건(Condition)이 모두 만족 시 발생합니다.
- 한 가지 조건(Condition)이라도 만족하지 않을 시 교착 상태(Deadlock)는 발생하지 않습니다.

  - ##### 2.1 상호 배제(Mutual exclusion)
    - 자원(Resource)은 한 번에 한 프로세스(Process)만 사용할 수 있습니다.
  - ##### 2.2 점유 대기(Hold and wait)
    - 최소한 하나의 자원(Resource)을 점유하고 있으면서 해당 자원(Resource)은 다른 프로세스(Process)에 할당되어 사용하고 있는 자원(Resource)을 추가로 점유하기 위해 대기하는 프로세스(Process)가 존재해야 합니다.
  - ##### 2.3 비선점(No preemption)
    - 다른 프로세스(Process)에 할당된 자원(Resource)은 사용이 끝날 때까지 강제로 빼앗을 수 없습니다.
  - ##### 2.4 순환 대기(Circular wait)
    - 프로세스(Process)의 집합에서 순환 형태로 자원(Resource)을 대기하고 있어야 합니다.

<br />


---

### 3. Solution For Deadlock

- 예방 & 회피(Prevention & Avoidance)
- 탐지 & 회복(Detection & Recovery)


- #### 3.1 Deadlock Prevention & Avoidance

  - ##### 3.1.1 예방(Prevention)
    - 교착 상태(Deadlock) 발생 조건 중 하나를 제거하면서 해결합니다
    - 자원(Resource)가 많이 필요해서 자원(Resource) 낭비 엄청 심합니다.
      - ###### 상호 배제(Mutual exclusion) 부정
        - 여러 개의 프로세스(Process)가 자원(Resource)을 사용할 수 있도록 합니다.
      - ###### 점유 대기(Hold and wait) 부정
        - 프로세스(Process)가 실행되기 전에 필요한 모든 자원(Resource)을 요청하고 허용되기까지 대기합니다.
      - ###### 비선점(No preemption) 부정
        - 우선순위(Priority)가 높은 프로세스(Process)에 대하여 다른 프로세스(Process)가 사용 중인 자원을 점유합니다. 
      - ###### 순환 대기(Circular wait) 부정
        - 자원(Resource)을 순환 형태가 아닌 일정한 방향으로만 요구할 수 있도록 합니다.

  - ##### 3.1.2 회피(Avoidance)
    - 주기적으로 교착 상태(Deadlock) 발생 가능성에 대하여 검사하여 회피하는 방식입니다.
    - 교착 상태(Deadlock)에 빠질 가능성이 없을 경우에만 자원(Resource)을 할당함으로써 문제 발생을 회피하는 방법입니다.
    - <mark style="background-color:#cccccc">Safe State & Unsafe State</mark>
![image info](/assets/img/safeUnsafe.png)
<img src="/assets/img/safeUnsafe.png" alt="" width="8" height="0">
      - Safe State
        - 교착 상태(Deadlock)가 일어날 가능성이 있습니다.
        - 프로세스(Process)가 요구한 양만큼 자원(Resource) 할당이 가능합니다.
        - 안전 순서열이 존재합니다.
      - Unsafe State
        - 교착 상태(Deadlock)가 일어날 가능성이 없습니다.
        - 프로세스(Process)가 요구한 양만큼 자원(Resource) 할당이 불가능합니다.
        - 안전 순서열이 존재하지 않습니다.
    - <mark style="background-color:#cccccc">은행원 알고리즘</mark>
      - 알고리즘의 의미
        - 은행은 모든 고객의 요구가 충족되도록 현금을 할당해야 하는 기법입니다.
      - 시스템상의 의미
        - 프로세스(Process)가 자원(Resource)을 요구할 때 시스템은 자원(Resource)을 할당한 후에도 안정 상태(Safe State)로 남아있게 되는지를 사전에 검사하여 교착 상태(Deadlock)를 회피하는 기법입니다.
      - 조건(Condition)
        - Max : 각 프로세스(Process)가 요청할 수 있는 최대 자원(Resource) 수 
        - Allocated : 각 프로세스(Process)가 현재 보유하고 있는 자원(Resource) 수
        - Available : 시스템가 보유하고 있는 자원(Resource) 수
    
<br />


- #### 3.2 Deadlock Detection & Recovery

  - ##### 3.2.1 탐지(Detection)
    - <mark style="background-color:#cccccc">자원 할당 그래프(Resource-Allocation Graph)</mark>
![image info](/assets/img/waitforgraph.png)
<img src="/assets/img/waitforgraph.png" alt="" width="0" height="0">
      - 자원 할당 그래프(Resource-Allocation Graph)를 통해 교착 상태(Deadlock)를 탐지함
      - 자원 할당 그래프(Resource-Allocation Graph)에서 프로세스(Process) 간의 대인 관계만 추출하면 wait-for graph를 얻을 수 있습니다.
      - 얻어진 wait-for graph를 통해서 교착 상태(Deadlock)인지 확인이 가능합니다.
        - <mark style="background-color:#cccccc">Resource-Allocation Graph</mark>
          - P->R는 프로세스(Process)가 자원(Resource)을 할당받기 위해 대기하고 있는 상태입니다.
          - R->P는 자원(Resource)이 프로세스(Process)에게 할당된 상태입니다.
        - <mark style="background-color:#cccccc">wait-for graph</mark>
          - 프로세스(Process)간의 대기 상태입니다.
          - wait-for graph에서 순환(cycle)이 발생하면 교착 상태(Deadlock)이 발생했다고 판단합니다.


  - ##### 3.2.2 회복(Recovery)
    - ###### 프로세스 종료(Process Termination)
      - 교착 상태(Deadlock)의 원인이 되는 프로세스(Process)를 모두 종료한다.
      - 프로세스(Process)를 하나씩 종료하면서 교착 상태(Deadlock)이 풀리는지 확인한다.
    - ###### 자원 선점(Resource Preemption)
      - 프로세스(Process)의 자원(Resource)을 선점하여 강제로 빼았습니다.
      - 희생 프로세스(Process)에 따라 희생 비용을 최소화합니다.
      - 자원(Resource)을 뺴앗긴 프로세스(Process)는 롤백(Rollback) 과정을 거치거나 처음 상태로 되돌려야 합니다.
      - 희생 프로세스(Process)의 기준에 따라 계속해서 같은 프로세스(Process)만 희생될 수 있습니다.
<br />


---

### Reference
- https://velog.io/@seorim0801/%EC%9D%80%ED%96%89%EC%9B%90-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98
- https://jwprogramming.tistory.com/12
- https://velog.io/@kangdev/%EA%B8%B0%EC%88%A0%EB%A9%B4%EC%A0%91%EC%9A%B4%EC%98%81%EC%B2%B4%EC%A0%9C-%EA%B5%90%EC%B0%A9-%EC%83%81%ED%83%9C-%ED%95%B4%EA%B2%B0-%EB%B0%A9%EB%B2%95Deadlock-Solution
- https://howudong.tistory.com/366