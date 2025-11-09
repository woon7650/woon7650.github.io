---
title: "ADOT & GSDM RAG Project(Document-level RAG Dataset Generation and Delivery Architecture Design)"
excerpt: "[Project Experience] ADOT Chatbot과 GSDM 연동 Project Data Pipline Architecture"
date: 2025-11-09
categories: [Java, Spring Boot, Spring Batch, Spring MVC, MyBatis, MSSQL, OnPremise, ExecutorService, CompletableFuture]
tags: [Java, Spring Boot, Spring Batch, Spring MVC, MyBatis, MSSQL, OnPremise, ExecutorService, CompletableFuture]
---

## Index

1. [들어가면서](#introduction)
2. [❌ Issue(Problem Definition)](#issue)
3. [⚠️ Potential Problem](#potential-problem)
4. [🔁 Consideration(Approach)](#approach)
5. [💡 TroubleShooting](#troubleshooting)
6. [✅ Conclusion](#conclusion)


---
### 들어가면서 {#introduction}

이번 프로젝트의 핵심 목표는 **ADOT Chatbot(RAG)을 통해 GSDM 문서 및 관련 정보를 조회하고 활용할 수 있도록 문서 단위 DataSet 생성과 Object Storage 업로드 처리 로직을 설계**하는 것이었습니다.  
GSDM 시스템은 이미 운영 중인 On-Premise 서버 환경에서 서비스되고 있어 기존 구조 변경이나 서버 증설이 어려웠습니다. 따라서 프로젝트 초기부터 **안정성 최우선, 성능과 확장성 고려, 향후 확장 가능성 확보**라는 세 가지 목표를 바탕으로 아키텍처 설계를 진행했습니다.

#### 핵심 목표
- **운영 서버 안정성 확보**: Batch, REST API, Message Queue 활용 시 기존 운영 시스템에 영향을 최소화하도록 설계했습니다.
- **대규모 데이터 처리 가능**: REST API는 AOP가 적용되지 않는 마이그레이션된 데이터 처리와 초기 데이터 생성 용도로 사용하며 5,000~10,000건 문서를 기준으로 로직 최적화를 수행했습니다.
- **문서 단위 DataSet 생성 자동화**: 문서 1건 단위로 JSON, Markdown, 첨부파일 세트를 생성하고 Object Storage에 업로드했습니다.
- **향후 확장성 확보**: 실시간 처리, 주기적 처리, 대규모 문서 처리를 기반으로 추가 요구 발생 시 기존 구조를 최소한으로 변경하여 확장 가능하도록 설계했습니다.

---

### ❌ Issue(Problem Definition) {#issue}

**1. DataSet 생성 요구 사항**  
- GSDM 문서 중 특정 조건을 만족하는 데이터에 대해 DB를 조회하고 Markdown(.md), JSON(.json) 파일과 최소 0개 이상의 첨부파일을 포함한 DataSet 단위로 Object Storage에 업로드해야 했습니다.  
- 하루 단위 처리 문서 건수는 100~5,000건 이상, 생성 파일은 300~15,000건 이상으로 대규모였습니다.  
- 문서 단위 생성 파일과 첨부파일 세트의 일관성을 위해 ADOT 개발자들과 I/F 설계 협업을 진행했습니다. JSON/Markdown 필드 매핑, directory 구조, 파일 명칭 매핑 등을 논의하여 Object Storage 내에서 Unique하게 확인할 수 있는 구조로 설계했습니다.  

**2. 서버 구조 및 제약**  
- 기존 GSDM WAS와 동일 서버에서 별도 포트를 활용한 Batch Server를 구축해야 했습니다. 서버 증설이 불가했기 때문에 단일 서버 내에서 안정적으로 운영될 수 있는 구조가 필수였습니다.  
- 프로젝트 기간은 1~2개월로 제한되었으며 기존 소스 변경은 최소화해야 했습니다.  
- 안정성을 최우선으로 하고, 대규모 데이터를 안정적으로 처리할 수 있는 구조를 설계했습니다.  

**3. 처리 방식 및 로직 흐름**  
- **REST API**:  
    - AOP가 적용되지 않는 마이그레이션 데이터와 초기 대상 데이터를 즉시 처리하기 위해 사용했습니다.  
    - 5,000~10,000건을 기준으로 대규모 처리 시 CPU/IO 분리, 비동기 병렬 처리로 최적화했습니다.  
    - 처리 완료 후 EventLog에 상태를 기록하여 Batch 및 MQ에서도 동일 로직 재사용 가능하게 구성했습니다.  
- **Batch**:  
    - EventLog 기반 주기 처리 용도로 사용하며, REST API에서 검증된 동일 로직을 Reader, Processor, Writer 형태로 재사용합니다.  
- **Message Queue(RabbitMQ)**:  
    - 향후 실시간 처리와 부하 분산을 위해 설계만 반영, REST API 최적화된 로직을 적용 예정입니다.  
- **공통 로직**: DB 조회(IO) → 내용 생성(CPU) → Markdown/JSON 생성(IO) + 첨부파일 복사(IO) → Object Storage 업로드(IO)  

---

### ⚠️ Potential Problem {#potential-problem}

1. **서버 영향 최소화**  
   - Batch, REST API, MQ 모두 동일 서버에서 실행될 수 있으므로 CPU와 메모리 점유율을 최소화하는 것이 중요했습니다.  

2. **대용량 I/O 비중 및 병목**  
   - 전체 처리 과정에서 파일 생성, 첨부파일 복사, 업로드 등 I/O 작업이 80~90%를 차지하며, CPU 연산 비중은 10~20% 정도였습니다.  
   - 단순 병렬화 또는 Future 기반 비동기 처리만으로는 오히려 처리 속도가 저하될 수 있어 CPU/IO 병목 분리, Executor Pool, Semaphore, ChunkSize, Future 수 등을 조정했습니다.

3. **확장성 고려**  
   - REST API를 기준으로 대규모 데이터 처리와 안정성을 검증했으므로 Batch와 향후 MQ에서도 동일 로직 재사용이 가능하도록 설계했습니다.  
   - EventLog를 통해 문서 단위 처리 상태(`completed`, `operation`, `version`, `document_id`)를 추적하여 장애 발생 시 재처리와 모니터링 가능성을 확보했습니다.

4. **대규모 데이터 처리 기반 설계**  
   - 소수 건수 기반이 아닌, 언제 얼마나 많은 문서가 들어올지 모르는 상황을 가정하여 설계했습니다.  
   - REST API → Batch → MQ 순서로 개발 진행하며, REST API에서 최적화된 로직을 검증 후 재사용함으로써 안정성과 확장성을 확보했습니다.

---

### 🔁 Consideration(Approach) {#approach}

**1. 아키텍처 설계**  
- GSDM 코드 변경 없이 AOP를 별도 파일로 구성하여 이벤트를 감지하고 EventLog 기반 문서 단위 처리를 수행하도록 설계했습니다.  
- 재사용 가능한 로직 적용: DB 조회(IO) → 내용 생성(CPU) → Markdown/JSON 생성 + 첨부파일 복사(IO) → Object Storage 업로드(IO)  
    - **REST API**: AOP가 적용되지 않은 마이그레이션 데이터 처리 및 초기 데이터 생성/즉시 처리 → 대규모 데이터를 기준으로 CPU/IO 분리 및 비동기 병렬 처리로 최적화  
    - **Batch**: REST API에서 검증된 로직을 Reader, Processor, Writer 형태로 재사용, 일정 주기 처리  
    - **RabbitMQ**: 향후 실시간 처리용, REST API 최적화된 로직 적용 예정  

**2. IF 설계 협업**  
- ADOT 개발자와 JSON/Markdown 매핑, `.json`, `.metadata`, 첨부파일 명 및 directory 구조를 협의했습니다.  
- Object Storage Bucket 및 directory 구조는 향후 확장성을 고려했고, 파일명은 Object Storage 내 유니크하게 realFileName 그대로 사용했습니다.  
- 이를 통해 RAG 처리 시 데이터 일관성과 호환성을 확보했습니다.

**3. 확장 고려**  
- 향후 실시간/주기 처리 확장을 위해 Listener/Worker 개념 적용 가능하도록 설계했습니다.  
- 재사용 가능한 로직 기반으로 Message Queue 도입 시 인프라 구축 외 기존 프로세스 변경 최소화 가능하도록 설계했습니다.

---

### 💡 TroubleShooting {#troubleshooting}

- 이번 포스트에서는 **아키텍처 설계와 판단 과정 중심**으로 서술했습니다.  
- 비동기/병렬 처리, I/O 병목 최적화 등 세부 구현 사항은 **별도 포스팅에서 상세 기술 예정**입니다.

---

### ✅ Conclusion {#conclusion}

이번 프로젝트를 통해 저는 **문서 단위 DataSet 생성 로직을 기반으로 REST API, Batch 구조 설계를 진행하고 적용**할 수 있었습니다.  

핵심 요약:
- 서버 영향 최소화 + 문서 단위 DataSet 생성 자동화  
- REST API, Batch 구조 모두 재사용 가능한 로직 사용  
- IF 설계 협업 반영 → JSON/Markdown/첨부파일 매핑, directory 구조 일관성 확보  
- REST API를 대규모 기준으로 최적화하여 안정성 확보, 향후 실시간 처리(MQ) 확장은 설계만 반영  

> 문서 단위 DataSet 생성 로직을 중심으로 REST API → Batch → MQ 순서로 개발하며 주기적, 실시간성, 예외 데이터 처리를 모두 고려한 안정적이고 확장 가능한 아키텍처를 설계했습니다.
