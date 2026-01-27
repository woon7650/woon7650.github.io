---
title: "Designing Hybrid RAG Architecture : ElasticSearch(instead Vector DB), Neo4j"
excerpt: "[AI, Java] ERP RAG 구축기(설계 편)"
date: "2026-01-26"
categories: [Backend, Infrastructure, AI]
tags: [ElasticSearch, Neo4j, GraphRAG, HybridSearch, VirtualThreads, SpringAI, WebFlux]
---

### 0. 들어가면서
* 특히 복잡한 비즈니스 로직과 방대한 문서를 다루는 ERP 시스템에서 단순한 Vector 검색은 `A 규정이 B 프로젝트의 특정 단계에 미치는 영향`과 같은 고차원적인 질문에 한계를 드러냅니다.
* Vector DB 대신 **ElasticSearch**의 통합 검색 능력과 **Neo4j**의 관계형 지능을 결합하여 이 한계를 극복한 **Hybrid RAG Architecture**의 설계 기록입니다. 왜 이 스택을 선택했는지 각 기술이 어떤 시너지를 내는지 및 아키텍처에 대해 정리해보고자 합니다.
* 초기 설계 단계로 변경/구축/최적화하는 과정을 기록할 예정입니다.

---

### 1. Tech Stack Strategy: 왜 이 기술들을 선택했는가?

아키텍처 설계의 본질은 자원의 효율적 배분과 비즈니스 요구사항의 완벽한 조화입니다. 인프라 비용 최적화와 검색 정확도라는 두 마리 토끼를 잡기 위해 초기에 다음과 같은 Tech Stack을 구성했습니다.

#### JDK 21 & Virtual Threads
* RAG 파이프라인은 LLM API 호출, 인덱싱 등 I/O Bound 작업이 90% 이상을 차지합니다. 
* 기존 Platform Thread는 하나당 약 1MB의 고정 스택 메모리를 점유하여 수천 개의 동시 I/O 처리 시 OOM을 초래합니다. 반면 Virtual Threads는 수 KB의 메모리만으로 실행 가능하며 대기 시 Heap으로 컨텍스트를 옮기는 Continuation 메커니즘을 통해 RAM 자원을 확보합니다. 
* 이렇게 확보된 여유 RAM이 **ElasticSearch와 Neo4j의 OS Page Cache**로 환원되어 검색 성능을 극대화하는 것을 목표합니다.

#### Spring WebFlux & Spring AI
* Netty의 Direct Buffer Pool을 활용하여 JVM Heap 외부 메모리에서 데이터를 직접 처리함으로써 **데이터 복사를 최소화(Zero-copy)** 하고 대용량 문서 인덱싱 시 발생하는 GC 오버헤드를 획기적으로 줄입니다.
* 여기에 Spring AI를 결합하여 Embedding 모델과 Vector 스토어에 대한 인터페이스를 표준하고 제공하는 다양한 기능들을 자세히 파악하고 활용하고자 합니다.

#### Neo4j
* ERP 데이터는 단순한 텍스트의 나열이 아닙니다. `부서 - 프로젝트 - 담당자 - 관련 규정`으로 이어지는 복잡한 **비즈니스 관계도**를 가집니다. 
* Vector 검색만으로는 찾을 수 없는 이 부분을 **Neo4j**를 통해 특정 규정을 찾았을 때 해당 노드로부터 **K-Hop Search(Graph Traversal)**을 수행하여 연결된 인접한 연관 지식을 함께 추출합니다. 이를 통해 검색의 범위를 단순 '유사성'에서 '관계 중심의 확장성'으로 넓혀 답변의 품질을 높이고자 합니다.

#### ElasticSearch 
* Milvus와 같은 전용 Vector DB는 '의미 유사도'에는 강하지만 특정 품번이나 정확한 고유명사 검색에는 취약합니다. ElasticSearch를 선택하여 **역인덱스의 정밀함과 Vector의 유연함**을 동시에 확보해서 오타가 있거나 추상적인 질문에는 Vector 검색이 답하고 명확한 키워드에는 역인덱스가 답하게 하여 Milvus 단독 사용 시보다 훨씬 높은 검색 정확도를 목표합니다.

---

### 2. Storage Architecture: Why ElasticSearch instead Milvus?

#### 필터링 성능의 한계 극복
* Milvus와 같은 Vector DB는 대규모 Vector 연산에는 강점이 있으나 비Vector 속성(부서, 생성일, 권한 등)에 대한 복합 필터링 시 성능이 저하되거나 쿼리가 복잡해지는 단점이 있습니다. 
* ElasticSearch는 이미 검증된 역인덱스 기술을 통해 Metadata 필터링과 Vector 검색을 단일 요청 내에서 최적으로 결합합니다.

#### HNSW (Hierarchical Navigable Small World)
* ElasticSearch는 고성능 ANN 검색을 위해 HNSW Algorithm을 사용합니다. 이는 Vector를 계층적 그래프로 연결하여 상위 층의 지름길을 통해 유사한 그룹으로 빠르게 접근하게 함으로써 수억 건의 데이터에서도 밀리초 단위의 검색 속도를 유지하게 합니다.

--- 

### 3. Data Processing : Hierarchical Chunking & Metadata Enrichment

#### Markdown-Aware Splitting 
* 단순히 글자 수로 자르지 않고 마크다운의 헤더 구조(`#`, `##`)를 구분자로 사용합니다. 소주제 단위로 잘려야 Chunk 자체가 독립적인 의미를 가집니다.

#### Context Injection 
* 잘린 조각만으로는 원본 문서의 전체 내용을 알 수 없습니다. 우리는 각 조각에 **[부모 문서 제목 + 상위 헤더 제목 + 파일 경로]** 를 Metadata로 강제 주입하여 Embedding합니다. 이를 통해 검색된 조각이 LLM에게 전달될 때 정보의 파편화를 방지합니다.

#### Overlap Strategy 
* 조각 간의 정보 단절을 방지하기 위해 이전 조각의 마지막 약 10~15%를 다음 조각에 포함시키는 Overlap을 설정하여 문장의 연속성을 유지합니다.

--- 

### 4. ElasticSearch : Hybrid Search (Lexical + Vector)

#### Lexical Search
* **BM25 Algorithm**을 사용하여 특정 고유명사나 전문 용어를 역인덱스(Inverted Index)에서 즉시 찾아냅니다.
* 의미적 유사성보다 **정확한 단어의 존재 여부**가 중요할 때 압도적인 신뢰도를 제공합니다.

#### Vector Search
* **HNSW(Hierarchical Navigable Small World) Algorithm**을 사용하여 수억 개의 Vector를 전체 탐색하는 대신 계층적 그래프를 타고 내려가며 질문과 가장 가까운 좌표의 문서를 **ANN(Approximate Nearest Neighbor)** 방식으로 탐색합니다.
* `휴가 규정이 궁금해`와 같이 단어가 직접 노출되지 않아도 맥락이 유사한 문서를 찾아내는 **의미 기반 검색**이 가능합니다.

#### Reciprocal Rank Fusion
* 어휘 기반 검색(BM25)과 의미 기반 검색(kNN)의 결과 리스트는 점수 산정 방식이 완전히 다릅니다. 이를 단순 합산하는 대신 RRF Algorithm을 통해 각 결과의 순위에 역수를 취해 병합합니다. 
* 해당 과정은 두 검색 방식의 '상호 보완'을 수학적으로 완성하는 단계입니다.

---

### 5. Neo4j : Relational Search (Knowledge Connectivity)

#### Value Of Relation
* Markdown 문서 내의 참조 링크나 문서 간의 계층 구조를 Node와 Edge로 구성합니다.

#### Graph Augmented Retrieval 
* 사용자가 `신입 사원 혜택`을 질문하면 시스템은 ElasticSearch로 `휴가 규정`을 찾고 **Neo4j**를 통해 그와 연결된 `사내 복지 포인트`, `업무 장비 지원` 등 **인접한 연관 지식**을 함께 추출합니다. 이는 검색의 범위를 단순 유사성에서 관계 중심의 확장성으로 넓혀줍니다.

---

### ✅ Conclusion

*  다음 포스팅에서는 설계한 Architecture의 개선점이나 초기 구축하는 과정을 진행하면서 정리하고자 합니다. 현재 프로젝트를 진행하면서 변경이 필요한 부분들, 내용상 부족/잘못 작성된 부분들은 바로 추가/반영할 예정입니다.