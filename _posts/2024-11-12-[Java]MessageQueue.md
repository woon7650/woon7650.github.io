---
title: "Asynchronous Messaging Pattern With Message Queues and Pub/Sub"
excerpt: "[Java] About Message Queues and Pub/Sub"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2024-11-10
---

##### RELATED POST
- [Java CompletableFuture](blog/Java-CompletableFuture/)
- [Spring Asynchronous](blog/Spring-SpringAsync/)

<br />

---

> ### MOM(Message Oriented Middleward)

  - 독립된 서비스 간 데이터를 주고 받을 수 있는 형태의 미들웨어
  - 메세지 API를 통해 분산되어 있는 Application 간의 데이터를 교환할 수 있도록 해주는 시스템
    - 비동기 메시지를 사용하는 Application 사이에서 데이터를 송수신
  - 시스템 간의 Connector 역할로 결합성을 낮추고 실시간 비동기식 데이터 교환이 가능


  - #### Architecture

    ![image info](/assets/img/MOM.png)
    <img src="/assets/img/MOM.png" alt="" width="0" height="0">

    - Queue, Broadcast, Multicast 방식으로 비동기 방식으로 메세지를 전달
    - Components
      - Publisher(Producer) : 메세지를 발행
      - Subscribe(Consumer) : 메세지를 소비
      - Message Broker : Message의 유효성, 전송, 라우팅을 위한 Architecture Pattern으로 Publisher로부터 전달받은 Message를 동일한 Topic의 Subscriber로 전달해주는 중간 역할
      - Message Queues : Message가 적재되는 공간
      - Topic : Message의 그룹


  - #### Asynchronous Messaging Pattern
    - Publisher(Producer)와 Subscribe(Consumer)를 분리하는 방식
      - Message Queues, Pub/Sub
      - 확장성과 안정성, 성능 향상이 가능

<br />

---

> ### MQ(Message Queues)
    

  - JMS(Java Message Service) 개방형 표준을 구현하는 표준 기반 메시징 솔루션
  - 메시지 지향 미들웨어(Message Oriented Middleware)를 구현한 시스템
  - Components
    - Producer : 메세지를 발행
    - Consumer : 메세지를 소비

  - #### Process

    ![image info](/assets/img/MQPUBSUB.png)
    <img src="/assets/img/MQPUBSUB.png" alt="" width="0" height="0">
 

    - Message Queues에는 Publisher가 넣은 n개의 메세지가 존재한다
    - Consumer A가 Message m1를 소비했기 때문에 Consumer B에서는 Message m1를 소비할 수 없다
    - Consumer B는 Message m2를 소비할 수 있다
    - **Message Queues는 Message 처리 작업을 Consumer에게 위임하며 Message에 대하여 작업을 한 번만 실행하도록 할 수 있다**

  - #### Features
    - 비동기(Asynchronous) : Message Queues를 통한 비동기 방식 지원(Consumer가 필요할 때 처리)
    - 낮은 결합도(Decoupling) : Producer, Consumer의 분리를 통한 결합도 감소
    - 회복 탄력성(Resilience) : 일부가 실패 시 전체에 영향을 주지 않음
    - 과잉(Redundancy) : 실패할 경우 재실행 가능
    - 보장성(Guarantees) : 큐에 있는 모든 메세지는 결국 Consumer에게 전달된다는 보장 제공
    - 확장성(Scalable) : 다수의 프로세스들이 큐에 메세지 전송 가능

  - #### Usage

    - Application 또는 System 간의 통신
      - 서버 간에 데이터를 주고 받을 때 시스템 장애를 염두해야 한다
      - 해당 Consumer가 Message를 받을 수 없는 경우
        - 해당 Conumser가 해당 메세지를 받을 때까지 MQ에 머물러 있거나 다른 Consumer에서 해당 메세지를 처리한다
    - 서버 부하가 많은 작업
      - 메모리, CPU를 많이 사용하는 작업은 동시에 처리하는 양이 한정적이다
      - MQ를 통해 동시에 처리할 수 있는 양에 따라 하나의 작업이 끝나면 다음 작업을 MQ에서 가져와 순차적으로 처리한다
    - 부하 분산
      - 여러 대의 서버가 하나의 MQ를 바라보도록 구성하면 처리할 데이터가 많아도 자신의 처리량에 맞게 테스크를 가져와 처리 가능하다
    - 데이터 손실 가능
      - MQ로 부터 가져온 업무를 일정 시간이 지나도록 처리했다고 다시 알려주지 않으면 MQ는 다시 큐에 넣어 다시 처리할 수 있도록 한다


  - #### OpenSource Message Queues
    - RabbitMQ, ActiveMQ, ZeroMQ, Kafka


<br />

---

> ### Pub/Sub

  - 하나의 Message는 하나의 Topic에 발행되고 Topic을 구독하는 모든 Subscriber는 Message의 사본을 받는다
  - Components
    - Publisher : 메세지를 발행하는 주체로 Subscribers를 직접 알 필요없이 특정 Channel/Topic에만 메시지를 전송한다
    - Subscriber : 메세지를 소비하는 주체로 Channel/Topic에 전달된 메시지를 받아 처리하고 Publisher를 직접 알 필요 없이 필요한 메시지가 오면 작업을 한다
  - 낮은 결합도 : Publisher, Subscriber가 서로에 대한 정보를 알 필요가 없다

  - #### Process

    ![image info](/assets/img/MQPUBSUB2.png)
    <img src="/assets/img/MQPUBSUB2.png" alt="" width="0" height="0">
 

    - Publisher가 Topic에 새로운 Message를 전송한다
    - Subscriber A, B는 모두 Message m1를 소비한다
    - Topic은 새로운 Message를 모든 Subscriber에게 복사하여 분배한다
    - **모든 Subscriber들이 Message의 복사본을 가지도록 보증한다**

  - #### Subscription Pattern
  
    - Ephemeral Subscription(일시적인 구독)
      - Consumer가 구독 중일 경우에만 구독 가능
      - Consumer가 다운될 경우 구독 및 아직 처리되지 않은 메세지 유실
    - Durable Subscription(지속적인 구독)
      - 구독은 명시적으로 제거되지 않으면 유지
      - Consumer가 다운될 경우 구독을 유지시키고 추후에 다시 메세징 처리가 가능


<br />

---


> ### MQ vs Pub/Sub

  - Consume(Subscriber)가 다운될 경우
    - Message Queues : 다른 Consumer가 Message를 대신 관리할 수 있다
    - Pub/Sub : 손실된 Message를 복구하면 Topic에서 사용할 수 있다
  - 핵심 기준
    - **모든 Consumer들이 모든 Message를 전부 수신해야 하는가?**

> ### Comparsion

  - Kafka(Pub/Sub Pattern)
    - 생산자 중심적인 설계로 구성되어 **생성자가 원하는 각 메시지**를 게시할 수 있도록 하는 메시지 배포 패턴
  - RabbitMQ(Message Broker Pattern)
    - 브로커 중심적인 설계로 구성되어 **지정된 수신인에게 메시지**를 확인, 라우팅, 저장 및 배달하는 역할을 수행하며 보장되는 메시지 전달

  - #### TPS
    - Message Queue별 Producer 성능
      ![image info](/assets/img/MQ.png)
      <img src="/assets/img/MQ.png" alt="" width="0" height="0">

    - Message Queue별 Consumer 성능
      ![image info](/assets/img/MQ2.png)
      <img src="/assets/img/MQ2.png" alt="" width="0" height="0">

    - Kafka : 100,000TPS~
    - RabbitMQ : 20,000TPS
    - 그래프 분석
      - 대용량 메세지 처리와 분산을 추구 한다면 Kafka >>> RabbitMQ
      - 뭐든 과하면 부작용이 있듯이 성능이 좋다고 무작정 Kafka를 사용하는 것이 아니고 규모와 시스템의 용도에 맞게 선택하는 것이 좋을 것이다


<br />

---

### 마무리하면서

- 다음 포스트에서는 Kafka와 RabbitMQ에 대해서 자세히 알아보고 현재 학습 중인 Elasticsearch Stack, Redis와 연계하여 학습하고 실습을 진행해볼 예정입니다.

### Reference

- https://velog.io/@itonse/%EB%A9%94%EC%8B%9C%EC%A7%80-%ED%81%90-vs-pubsub%EB%B0%9C%ED%96%89%EA%B5%AC%EB%8F%85
- https://escapefromcoding.tistory.com/706
- https://sunrise-min.tistory.com/entry/Message-Queue-Message-Broker%EC%99%80-pubsub-%EC%B0%A8%EC%9D%B4?category=1059769
- https://12bme.tistory.com/176
- https://willseungh0.tistory.com/32
- https://jh-yoon.tistory.com/14
- https://docs.kakaocloud.com/service/analytics/pub-sub/pub-sub-overview
- https://joojae.com/what-is-message-queue/
- https://yunchan97.tistory.com/84
- https://velog.io/@joosing/create-flexible-delivery-structure-with-message-queue