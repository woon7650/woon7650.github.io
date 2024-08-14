---
title: "MSA(Microservice Architecture)"
excerpt: "[Architecture] MSA part1"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2024-08-14
---

#### 0. 들어가면서

> In software engineering, a microservice architecture is an architectural pattern that arranges an application as a collection of loosely coupled, fine-grained services, communicating through lightweight protocols. One of its goals is to enable teams to develop and deploy their services independently. This is achieved by reducing several dependencies in the codebase, allowing developers to evolve their services with limited restrictions, and hiding additional complexity from users. Consequently, organizations can develop software with rapid growth and scalability, as well as use off-the-shelf services more easily.

> 소프트웨어 엔지니어링 에서 마이크로 서비스 아키텍처는 애플리케이션을 느슨하게 결합되고 세분화 된 서비스 의 컬렉션으로 구성하여 가벼운 프로토콜을 통해 통신하는 아키텍처 패턴 입니다. 목표 중 하나는 팀이 독립적으로 서비스를 개발하고 배포 할 수 있도록 하는 것입니다. 이는 코드베이스에서 여러 종속성을 줄이고 개발자가 제한된 제한으로 서비스를 발전시킬 수 있도록 하며 사용자에게 추가적인 복잡성을 숨김으로써 달성됩니다. 결과적으로 조직은 빠르게 성장하고 확장 가능한 소프트웨어를 개발할 수 있을 뿐만 아니라 기성형 서비스를 보다 쉽게 ​​사용할 수 있습니다.

- 과거 프로젝트에서는 Monolithic Architecture만 다뤄왔지만 요새 트랜드에 맞춰서 Microservice Architecture에 대해서 알아보고자 합니다.
- 최근에 대용량 트래픽 처리에 관심이 생기면서 MSA와 Kafka에 대해서 알아보고자 합니다.
- 기회가 된다면 MSA 기반 토이 프로젝트도 만들어 볼 생각입니다.

<br />

---

### 1. Microservice Architecture & Monolithic Architecture

- #### 1.1 Monolithic Architecture(MA)

  - Application의 모든 구성 요소가 한 프로젝트에 통합되어 있는 Architecture
      
  - ##### 1.1.1 FlowChart

    ![image info](/assets/img/Monolithic.png)
    <img src="/assets/img/Monolithic.png" alt="" width="0" height="0">

  - ##### 1.1.2 Pros & Cons

    - Pros
      - 소규모 프로젝트에 대하여 개발 초기에 개발이 용이
      - 하나의 서버 및 DB 환경이기 때문에 환경을 고려한 개발이 쉬움
      - End-to-End 테스트가 용이

    - Cons
      - **작은 수정 사항도 전체를 다시 빌드하고 배포해야 함**
      - **일부분의 오류가 전체에 영향을 미침**
      - Service의 크기와 비례하여 시스템이 무거워짐
      - 기술 스택이 한번 정해지면 바꾸기 어려움
      - 부하 분산을 위해 각 컴포넌트를 독립적으로 확장하기 어려움



- #### 1.2 Microservice Architecture(MSA)

  - 하나의 큰 Application을 여러 개의 작은 Application으로 조개어 변경과 조합이 가능하도록 만든 Architecture
  - 하나의 큰 Service -> 경량화되고 독립적인 여러 개의 Service로 모듈화

  - ##### 1.2.1 FlowChart

    ![image info](/assets/img/MicroService.png)
    <img src="/assets/img/MicroService.png" alt="" width="0" height="0">

  - ##### 1.2.2 Feature

    - 각각의 Service는 크기가 작을 뿐, Service 자체는 하나의 Monolithic Architecture와 유사한 구조를 갖음
    - 각각의 Service는 독립적으로 배포가 가능해야 함
    - 각각의 Service는 다른 Service에 대한 Dependency가 작아야 함
    - 각각의 Service는 개별 프로세스로 구동되며, REST API와 같은 가벼운 방식으로 통신되어야 함

  - ##### 1.2.3 Pros & Cons

    - Pros
      - 배포 관점 : 
        - **Service별 개별 배포가 가능**(전체 서비스 중단 없음)
        - 독립 배포가 가능함 -> 개발자의 자율성이 증가
        - 요구사항만을 신속하게 반영하여 빠르게 배포 가능
      - 확장 관점 : 
        - 특정 서비스에 대한 확장성이 용이함
        - 클라우드 사용에 적합
      - 장애 관점 : 
        - 장애가 전체 서비스로 확장될 가능성이 적음
        - 부분적 장애에 대한 격리가 수월(장애가 일어난 Service에 대하여 적용)
      - 코드/유지보수 관점 : 
        - 팀 별로 프로젝트가 분리되어 있어서 코드의 이해도가 증가하고 유지보수가 쉬움
      - 스택 관점 : 
        - 각 Service에 맞는 언어와 프레임워크를 선택할 수 있음


    - Cons
      - 성능 관점 : 
        - Service간 호출 시 API를 사용하기 때문에 통신 비용 및 지연시간 증가
      - 데이터 관리 관점 : 
        - 데이터가 여러 Service에 분산되므로 한 번에 조회가 어렵고, 데이터의 정합성 관리가 어려움
      - 테스트/트랜잭션 관점 : 
        - 단위 테스트는 쉽지만, 통합 테스트 및 End-to-End 테스트 단위 시 여러 Service의 API를 검증해야 되서 시간과 비용이 많이 듬
      - 복잡성 관점 : 
        - Architecture가 다소 복잡해서 개발 및 관리가 어렵고 비용이 많이 듬


<br />

---

### 2. Best Choices For Business


  - Project Case
    - 초기 MA(Monolithic Architecture)로 구축하고 MSA(Microservice Architecture)로 전환
    - 초기 MSA(Microservice Architecture)로 구축
    - 초기 MA(Monolithic Architecture)로 구축하고 그대로 Service 유지

  - #### 2.1 Criteria

    ![image info](/assets/img/ma_msa.png)
    <img src="/assets/img/ma_msa.png" alt="" width="0" height="0">



  - #### 2.2 Consideration
    - 비용 :
      - MSA를 도입할 경우 비용 절감을 할 수 있는가?
    - 생산성 : 
      - MSA를 요구할 만큼 시스템 복잡도가 높은가?
      - 복잡도를 높인 MSA가 생산성을 저해하지 않는가?
    - 운영 : 
      - 개발, 운영을 동시에 할 만큼 인프라가 준비되었는가?
      - 인력이 MSA를 관리할 역량이 되는가?
    - 배포 : 
      - 배포를 충분히 자주하고 있는가?


<br />

### 마치며

최근 MSA가 대세이지만 시스템의 Architecture는 비즈니스 상황을 고려해서 선택해야 됩니다. 개발 과정에 항상 맞는 답은 없지만 해당 기준들을 바탕으로 결정한다면 Application에 더 적합한 Architecture를 결정할 수 있을 것입니다. 다음 글에서는 MSA에 대해서 자세히 알아보겠습니다.


---
### Reference

- https://www.inflearn.com/pages/infcon-2023-tech-msa?srsltid=AfmBOopKbvu18GgV8Zf4-Ehkonz-iS0PQAoGB9zkw1pw2eF58n73jceQ
- https://steady-coding.tistory.com/595
- https://doqtqu.tistory.com/328
- https://velog.io/@heoseungyeon/MSA-vs-%EB%AA%A8%EB%86%80%EB%A6%AC%EC%8B%9D-akg64flw