---
title: "Caching Strategy"
excerpt: "[Spring] Caching Strategy"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2024-10-17
---


> ### Caching

  - 데이터를 빠르게 접근하기 위해 Memory에 데이터를 임시로 저장하는 기술이다. DB나 Disk 보다는 빠른 Memory를 사용하여 데이터를 검색하는 방법이다.
  - 자주 사용하거나 변동이 적은 데이터를 Memory에 저장해두고 사용하면 속도를 크게 향상시킬 수 있다.
  - 사용자 경험을 개선하고 서버의 부하를 줄이는 데 큰 역할을 합니다.
  - Caching을 구현할 때는 다양한 전략을 사용할 수 있다.(Memory Caching, Disk Caching, Distributed Caching..)
  - Caching된 데이터가 오래되면 정확하지 않은 데이터를 제공하기 때문에 적절한 무효화 전략을 통해 최신 데이터를 유지해야 한다.
  - DB에 접근을 최소화, 네트워크 지연 시간 감소, 서버의 부하를 분산 시키는 효과가 있다.


<br />

---

> ### Caching Strategy

  - 웹 서비스 환경에서 시스템 성능 향상을 위해 가장 중요한 기술이다.
  - Caching Strategy에는 아래와 같은 전략들이 존재한다.
  - Category
    - Cache Aside
    - Read-Through
    - Write-Through
    - Write-Around
    - Write-Back

  - #### Consideration

    - Application 특성, 데이터 특성, Traffic 패턴 등을 고려해야 한다.
    - Caching은 데이터의 신선도와 일관성을 유지하는 것과 성능 사이의 균현을 맞추는 작업이다. 따라서 잘못 구현된 Caching Strategy는 오히려 성능 저하나 데이터 일관성 문제를 초래한다.
    - Cache의 크기, 만료 정책, 데이터 일관성 유지 방법을 신중하게 고려해야 한다.
    - 데이터의 변경 주기
      - 변동이 적은 데이터를 Caching할 경우 긴 만료 시간 설정
      - 변경이 잦은 데이터를 Caching할 경우 짧은 만료 시간을 설정, 데이터 변경 시 즉시 갱신할 필요가 있다.


<br />

---

> ### Cache Aside

  - 가장 많이 사용되는 Caching Strategy
  - Cache를 옆에 두고 필요할 때만 데이터를 캐시에 로드하는 전략
  - Cache는 별도 관리되고 의존성이 낮다.
  - 읽기가 많은 작업(Read-Heavy Workloads)에 적합하다.
  - Redis, Memcached

  - #### FlowChart

    ![image info](/assets/img/CacheAside.png)
    <img src="/assets/img/CacheAside.png" alt="" width="0" height="0">

  - #### Process

    - Cache에 데이터 존재(Cache Hit)
      - Cache에서 조회된 데이터를 제공한다.
    - Cache에 데이터 미존재(Cache Miss)
      - Application에서 DB로 데이터를 조회한다.
      - Cache에 데이터를 저장한다.
      - 사용자에게 데이터를 제공한다. 


  - #### Pros

    - 실제로 사용되는 데이터를 Caching하기 때문에 자원을 효율적으로 사용가능하다.
    - Cache Error(Cache Cluster Down)가 생겨도 DB에서 데이터를 조회하기 때문에 시스템 전체의 오류로 전파되지 않는다.(탄력적)
    - 로직에 따라 DB의 데이터 모델과 다른 형태로 Cache에 저장이 가능햐댜.

  - #### Cons
    
    - Cache에 없는 데이터인 경우 시간이 더 오래 걸린다.
    - 데이터 일관성이 깨질 수 있다.
      - Cache에서 데이터가 없을 때만 DB에서 데이터를 조회하여 Cache에 저장하기 때문에 동기화 문제가 발생할 수 있다.



<br />

---

> ### Read-Through

  - Cache Aside와 비슷한 Strategy
  - Application, Cache(주체), DB를 **inline**으로 배치
  - **읽기가 많은 작업(Read-Heavy Workloads)에 적합하다**.
  - Cache Aside와 달리 DB와 데이터의 모습이 동일하다.


  - #### FlowChart

    ![image info](/assets/img/Read-Through.png)
    <img src="/assets/img/Read-Through.png" alt="" width="0" height="0">

  - #### Process

    - Cache에 데이터 존재(Cache Hit)
      - Cache에서 조회된 데이터를 제공한다.
    - Cache에 데이터 미존재(Cache Miss)
      - Cache에서 DB로 데이터를 조회한다.
      - Cache에 데이터를 저장한다.
      - 사용자에게 데이터를 제공한다. 


  - #### Pros

    - 읽기가 많은 작업(Read-Heavy Workloads)에 적합하다.

  - #### Cons
    
    - 처음 Request는 무조건 Cache Miss가 발생한다.
      - Batch, Scheduler를 통해서 Cache에 데이터를 미리 적재하기도 한다.

<br />

---

> ### Write-Through

  - Read Through와 반대로 구성
  - 데이터를 DB에 저장할 때마다 Cache에 데이터를 추가하거나 업데이트한다.
  - Cache는 항상 최신 상태의 데이터를 유지한다.
  - Read-Through와 함께 사용하면 Read-Through의 모든 이점을 얻으면서 데이터의 일관성을 보장 받을 수 있다.
    - AWS DynamoDB Accelerator(DAX)

  - #### FlowChart

    ![image info](/assets/img/Write-Through.png)
    <img src="/assets/img/Write-Through.png" alt="" width="0" height="0">

  - #### Process

    - Cache에 이미 데이터가 있는지 여부에 상관없이 Cache에 데이터를 저장한다.
    - DB에 데이터를 저장한다.

  - #### Pros

    - **Cache, DB의 데이터가 항상 동기화되어 있다**.
    
  - #### Cons
    
    - 사용하지 않는 데이터도 Cache에 저장된다.(리소스 낭비)
    - 쓰기 지연 시간이 증가한다.

<br />

---

> ### Write-Around

  - DB에 직접 기록되며 읽은 데이터만 Cache에 저장된다.
  - Write 시에 Cache를 거치지 않고 DB에 기록된다.
    - Application -> DB -> Cache(읽은 데이터만)
  - 기록하는 데이터가 자주 사용되지 않는 경우 적합하다.

  - #### FlowChart

    ![image info](/assets/img/Write-Around.png)
    <img src="/assets/img/Write-Around.png" alt="" width="0" height="0">

  - #### Process

    - Cache에 데이터 존재(Cache Hit)
      - Cache에서 조회된 데이터를 제공한다.
    - Cache에 데이터 미존재(Cache Miss)
      - Cache에서 DB로 데이터를 조회한다.
      - Cache에 읽어온 데이터만 저장한다.
      - 사용자에게 데이터를 제공한다. 

<br />

---


> ### Write-Back(Write-Behind)

  - Application은 즉시 확인하는 Cache에 데이터를 먼저 저장하고 지연 후에 데이터를 DB에 저장한다.
  - 쓰기가 많은 작업(Write-Heavy Workloads)에 적합하다.
  - Cache에 먼저 데이터를 저장 -> 특정 서비스가 특정 주기로 Cache의 데이터 DB에 저장
  - Application에서는 Cache에만 Write 요청 처리
    - 별도의 서비스(Batch, Scheduler)을 통해 DB에 데이터를 동기화한다.(async)
  - **Read-Through & Write-Back**
    - 아주 짧은 시간 대량의 트래픽으로 순식간에 처리해야 하는 경우 유용하다.

  - #### FlowChart

    ![image info](/assets/img/Write-Back.png)
    <img src="/assets/img/Write-Back.png" alt="" width="0" height="0">

  - #### Process

    - Cache에 이미 데이터가 있는지 여부에 상관없이 Cache에 데이터를 저장한다.
    - DB에 데이터를 저장한다.

  - #### Pros

    - 대량의 Write 요청이 있는 경우 적합하다.
    - Write 비용을 절약한다.
    
  - #### Cons
    
    - Cache에 문제가 생길 경우 데이터 유실에 따른 문제가 발생할 수 있다.
      - Cache에 데이터를 모았다가 DB에 특정 주기에 저장한다.



<br />

---



### 마무리하면서

- Caching을 구현할 때 다양한 전략을 사용할 수 있습니다. 각각의 전략에는 장단점이 있으며 상황에 따라 맞는 조합으로 사용해야합니다. 서비스의 기획 의도나 로직에 따라서 Read와 Write Strategy를 효율적으로 사용하는 것이 올바르며 백엔드 개발에서 성능 향상 및 리스크에 대한 감소는 필수적으로 고려해야 하는 부분이기 때문에 Caching을 사용하게 된다면 각각의 전략의 장단점에 대해 파악하고 있는 것이 좋다고 생각합니다.

---

### Reference

- https://velog.io/@psk84/Caching-%EC%A0%84%EB%9E%B5
- https://f-lab.kr/insight/caching-in-backend-development-20240706
- https://wnsgml972.github.io/database/2020/12/13/Caching/
- https://f-lab.kr/insight/caching-strategy-and-data-optimization