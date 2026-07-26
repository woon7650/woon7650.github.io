---
title: "[Backend] Large-Scale Traffic Engineering — Intro"
excerpt: "[Intro] Purpose & Roadmap"
date: "2026-07-26"
categories: [Backend, Series]
tags: [LargeScaleTraffic, MSA, DistributedSystem, Backend, StudyLog]
toc: true
toc_sticky: true
toc_label: "목차"
last_modified_at: 2026-07-26
---

### 시작하면서

#### 배경
실제로 다뤄온 프로젝트 중 일부는 사용자가 늘어나고 시스템이 커질수록 보완해야 할 부분이 하나씩 드러났습니다. SI 위주로 내부 시스템을 주로 다뤄오다 보니 초기·중반 규모까지의 경험은 쌓였지만 대규모까지 갖춰진 이후의 경험은 상대적으로 적었습니다. 실제 주변에서 자주 쓰는 서비스들 예매나 주문·거래 같은 서비스들이 지금 이 순간에도 얼마나 많은 트래픽을 어떻게 감당하고 있는지에 의문과 궁금증이 생겼습니다. 제가 다뤄온 환경과 실제 주변에서 자주 사용하는 서비스와의 차이를 파악하면서 서비스의 규모와 성격에 따라 어떤 아키텍처와 스택을 선택했는지 하나씩 비교하며 확인하기 시작했습니다.

#### 대응
스레드가 응답을 기다리는 동안 그대로 점유된 채 낭비되는 문제를 마주쳤습니다. 인프라를 늘릴 수 없는 상황이라 우선은 같은 서버 안에서 배치 작업처럼 운영 WAS에 부담을 주는 일과 JDK8에서는 쓸 수 없는 라이브러리가 필요한 일을 별도 WAS로 분리했습니다. 동기 방식으로 처리하는 이상 스레드 점유 문제 자체가 풀린 건 아니었지만 별도 프로세스라 메모리 영향과 장애·배포 영향은 서로 번지지 않게 됐습니다. 그 과정에서 장애 방어 장치와 캐시, 지표 모니터링까지 함께 다뤄봤습니다.

#### 한계
이 중에서도 스레드 문제만큼은 CompletableFuture로 비동기 처리를 시도해도 그 작업을 실행하는 스레드 자체는 여전히 필요했습니다. 대기 비중이 높은 작업들이라 코어 수가 천장은 아니었지만 그 대기 시간을 결국 스레드 수로 버텨야 하는 구조 자체가 한계였습니다. MVC 같은 동기 모델에서 Virtual Thread까지는 스택 조합에 맞게 자원 낭비를 줄이는 방향으로 시도해봤습니다. 앞으로는 트래픽 규모나 시스템 성격에 따라 MVC·JDBC 조합이 한계에 부딪히는 지점을 살펴보고 그 조건에 맞을 때 웹 계층은 Netty 기반 WebFlux로 DB 계층은 R2DBC로 바꾸는 것도 고려해보고 싶습니다.

#### 회고
서버 한 대에 장애가 생기면 그 위에 올라간 기능이 전부 함께 내려가는 위험은 그대로 안고 있었습니다. 그래도 이렇게 역할별로 프로세스를 나눠본 경험은 지나고 보니 MSA와 방향이 겹치는 부분이 있었다는 생각이 듭니다. 코어나 메모리까지 따로 확보하지는 못했고 닮은 건 스택을 독립적으로 가져갈 수 있었다는 정도까지였습니다. 다만 서비스를 실제로 나누는 구조로 나아가면 데이터베이스 역시 한 대로는 감당이 안 되는 시점이 오고 그때는 Read Replica나 Sharding처럼 데이터 자체를 나누는 것도 별도로 필요해진다는 것까지는 알고 있습니다. 여러 서버로 나뉘었을 때 생기는 동시성·정합성 문제나 장애 전파는 아직 직접 겪어본 적이 없어 기존 구조로는 근본적으로 풀리지 않던 문제를 진단하고 해결하는 트러블슈팅 역량을 채우고자 합니다.

#### 목표

이 시리즈는 무조건 새로운 걸 배우자는 게 아니라 직접 부딪혀서 느낀 이 공백을 정확히 채우기 위한 학습입니다. 목표는 트래픽이 몰리는 상황에서도 안정적인 아키텍처를 설계하고 SLO를 정의하고 최종적으로 달성하는 것입니다. 정합성과 동시성과 가용성이 부딪히는 여러 상황과 장애 상황에도 흔들림 없이 대응할 수 있는 역량을 갖추는 것도 목표입니다. 더 크게 보면 서비스가 초기·중반 규모에 머물 때뿐 아니라 확장되고 커진 이후까지 대응하며 운영할 수 있는 역량을 갖추고자 합니다.

이어지는 포스팅들은 아래 로드맵 순서대로, 하나의 문제 영역씩 원리 → 구현 → 트레이드오프 → 선택 기준 순으로 다룹니다.

---

### 1. 학습 순서

앞서 이야기한 자원 제약 속 운영 경험이 학습 순서에도 그대로 이어집니다. 하나의 서버 안에서 생기는 문제를 두 갈래로 먼저 봅니다. 데이터를 다루는 락·인덱스·트랜잭션과 그 데이터를 처리하는 스레드 모델·GC입니다. 그다음 여러 서버로 흩어졌을 때 생기는 분산 시스템 이론과 인프라 문제로 넘어간 뒤 서버를 역할별로 나누는 MSA 구조와 그 이후에 마주치는 확장성 문제까지 다루는 순서로 잡았습니다.

### 2. 학습·작성 방식

- **원리 중심**: DB 내부 구조, JVM 동작, 네트워크 레벨의 로우레벨 메커니즘까지 설명합니다. "무엇이다"에서 멈추지 않고 "왜 그렇게 동작하는가"까지 다룹니다.
- **트레이드오프 명시**: 모든 전략에는 장점만큼 단점과 한계도 함께 씁니다.
- **공식 문서 기반 검증**: MySQL, Redis, Kafka, Spring, Elasticsearch 등 공식 문서와 대조해 확인한 내용만 씁니다. 확인되지 않은 내용은 단정하지 않고 조건을 명시합니다.
- **경험 범위와 인지 범위 구분**: DB는 직접 다뤄온 MySQL InnoDB를 기준으로 깊게 쓰고 PostgreSQL처럼 구현이 다른 스택은 차이가 나는 지점만 비교로 짚습니다. 직접 겪은 것과 이해하고 있는 것을 섞지 않고 씁니다.

### 3. 전체 로드맵

| PART | 주제 | 다루는 내용 |
|---|---|---|
| PART 1 | 동시성 제어와 락 전략 | 비관적 락, 낙관적 락, Redis 분산락, 결제 방어 패턴 |
| PART 2 | Index & N+1 | B+Tree, 복합 인덱스, JPA N+1 해결 전략 |
| PART 3 | Transaction & MVCC | ACID, 격리 수준, InnoDB Undo Log·ReadView, PostgreSQL 비교 |
| PART 4 | 동시성 모델 | MVC(Thread-per-Request), Virtual Thread, WebFlux, R2DBC |
| PART 5 | JVM GC 튜닝 | G1GC·ZGC, Stop-the-World, 동시성 모델과의 상호작용 |
| PART 6 | CAP 정리 | CP·AP, PACELC, Quorum |
| PART 7 | 분산 트랜잭션 | 2PC, Saga, 멱등성 |
| PART 8 | Redlock & Fencing Token | 분산 락의 한계와 보완 |
| PART 9 | Redis 심화 | 캐싱 전략, 캐시 3대 문제, Sentinel·Cluster |
| PART 10 | Kafka 아키텍처 | Partition, ISR, Consumer Group |
| PART 11 | Kafka 신뢰성·실무 패턴 | 전달 보장, 멱등성 Producer, Outbox |
| PART 12 | Load Balancing & API Gateway | L4·L7 로드밸런싱, Spring Cloud Gateway |
| PART 13 | Service Discovery & 선언적 HTTP 클라이언트 | Eureka, Feign과 Spring Interface Clients 비교 |
| PART 14 | Circuit Breaker & Rate Limiting | Resilience4j, Token·Leaky Bucket, Sliding Window |
| PART 15 | 분산 트레이싱 | Trace ID·Span ID 기반 요청 추적 |
| PART 16 | API 설계 스타일 | REST vs gRPC vs GraphQL, 멱등성 설계 |
| PART 17 | 인증·인가 심화 | OAuth2, JWT 발급·갱신 전략 |
| PART 18 | 실시간 통신 | WebSocket |
| PART 19 | DB 성능 튜닝 & Sharding | HikariCP, Read Replica, 수평 분할, 분산 ID 생성 |
| PART 20 | Consistent Hashing | 해시 링, 가상 노드, 재분배 최소화 |
| PART 21 | NoSQL 데이터 모델링 | Cassandra·DynamoDB 설계 패턴 |
| PART 22 | 검색 인프라 | Elasticsearch, 역인덱스 |
| PART 23 | CDN | Edge 캐싱, Origin 관계, 캐시 무효화, Object Storage |
| PART 24 | 부하테스트·모니터링 | k6, Prometheus, Grafana |
| PART 25 | 배포 전략 | Blue/Green, Canary, Rolling |
| PART 26 | Auto Scaling | 지표 기반 스케일링, Scaling Policy |

### 4. 읽는 방법

순서대로 읽으면 데이터 정합성 → JVM·동시성 → 분산 시스템 → 캐싱·메시징 인프라 → MSA 아키텍처 → 확장성 인프라 순으로 이어지도록 구성했지만 각 포스팅은 독립적으로도 읽을 수 있게 썼습니다. 관심 있는 주제부터 먼저 봐도 무방합니다.

---

### ✅ Conclusion

이 시리즈는 정답을 알고 있어서 쓰는 글이 아니라 모놀리식에서는 보이지 않았던 문제들을 하나씩 확인하며 정리하는 기록입니다. 트래픽이 커질 때 드러나는 문제들은 대부분 새로운 지식이 아니라 이미 알려진 원리를 얼마나 정확히 이해하고 있느냐의 문제라고 생각하고 그 원리들을 공식 문서 기준으로 검증하면서 하나씩 포스팅으로 남기겠습니다.
