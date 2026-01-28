---
title: "Designing Hybrid RAG Architecture : ElasticSearch(instead Vector DB), Neo4j - part 2"
excerpt: "[AI, Java] ERP RAG 구축기(설계 편)"
date: "2026-01-27"
categories: [Backend, Infrastructure, AI]
tags: [ElasticSearch, RAG, HybridSearch, DocValues, InvertedIndex, HNSW, SpringAI]
---

### 0. 들어가면서 : VectorDB가 아닌 ElasticSearch ?

* 현대적 AI 애플리케이션의 핵심인 RAG 시스템은 단순히 '유사한 텍스트'를 찾는 수준을 넘어섰습니다. 기업용 데이터 환경에서는 **정확한 필터링, 수치적 범위 연산, 의미적 추론**이 동시에 정교하게 맞물려야 합니다.
* 전용 Vector DB가 `의미 유사도`라는 하나의 부분에 집중할 때 ElasticSearch는 **역인덱싱 기술과 최신 Vector 라이브러리(Lucene HNSW)** 를 통합하여 Hybrid Search의 표준을 제시하고 있습니다. 
* 본 포스팅에서는 **ElasticSearch의 로우레벨 아키텍처를 분석**하고 이를 어떻게 고성능 RAG 시스템으로 승화시킬 수 있는지를 정리해보고자 합니다.

---

### 1. ElasticSearch : Knowledge Storage Structure

하나의 문서가 인덱싱될 때 ElasticSearch는 이를 서로 다른 용도의 세 가지 구조로 분해하여 디스크/RAM에 저장합니다.

#### 1.1 역인덱스 (Inverted Index)

가장 전통적이고 강력한 구조입니다. 텍스트 데이터를 분석하여 `어떤 단어가 어느 문서에 있는지`를 사전 형태로 기록합니다.

* **기반 자료구조 :** FST(Finite State Transducer) 및 Skip List
* **매커니즘 :** Text를 단어 단위로 나눠서 '단어 → 문서 ID 리스트' 형태로 매핑합니다. FST를 통해 메모리 내에서 사전을 압축 관리하며 Skip List를 통해 문서 ID 간의 교집합 연산을 최적화합니다.
* **성과 :** 데이터가 늘어나도 검색 복잡도를 기존 O(N)에서 O(log N)이하로 유지하며 텍스트 일치 여부를 판단하는 데 있어 가장 효율적인 구조입니다.

#### 1.2 Doc Values

Lucene 4.0부터 도입된 구조로 `역인덱스와는 정반대의 형태`를 띱니다.

* **기반 자료구조 :** 열 지향 저장소 (Columnar Storage)
* **매커니즘 :** 역인덱스와 정반대로 '문서 ID → 필드 값' 구조로 저장합니다. 동일 필드의 데이터가 디스크 상에 연속적으로 배치됩니다.
* **OS Page Cache 및 Sequential I/O :** 데이터의 연속성 덕분에 OS 수준의 페이지 캐시 효율이 극대화됩니다. 정렬이나 집계 시 디스크의 여러 곳을 탐색하는 Random Read를 배제하고 Sequential Read를 수행하여 I/O 병목을 근본적으로 제거합니다.

#### 1.3 Dense Vector (HNSW)

ElasticSearch 8.x에 들어오며 RAG의 핵심이 된 `Vector 전용 구조`입니다. 단어가 일치하지 않아도 의미가 유사한 데이터를 찾을 때 효과적입니다.

* **기반 자료구조 :** HNSW (Hierarchical Navigable Small World) 그래프
* **매커니즘 :** 텍스트의 고차원 Vector 좌표를 저장하고 점들 사이의 근접성을 그래프 계층으로 연결합니다. 
* **ANN (Approximate Nearest Neighbor) :** 모든 Vector를 전체 탐색 `O(N)`하는 대신 상위 계층의 지름길 node를 거쳐 목표 지점 근처로 이동한 후 하위 계층에서 정밀 탐색을 수행합니다. 이를 통해 수억 건의 데이터에서도 ms단위의 유사도 검색을 보장합니다.

* **HNSW (Hierarchical Navigable Small World) :** 모든 Vector를 탐색하는 것은 `O(N)의 비용`이 들어 불가능합니다. ElasticSearch는 HNSW 그래프를 사용하여 Vector들을 계층적으로 연결합니다. 상위 계층에서 지름길을 찾아 빠르게 목표 근처로 이동한 뒤 하위 계층에서 **ANN(Approximate Nearest Neighbor)** 탐색을 통해 가장 가까운 이웃을 효과적으로 찾아냅니다.

---

### 2. Metadata Filtering : Pre-filtering Mechanism

RAG 시스템에서 Metadata Filtering은 단순한 검색 조건이 아니라 연산의 범위를 물리적으로 한정하여 **시스템의 예측 가능성**을 확보하도록 합니다.

#### 2.1 Pre-filtering 및 Bitset 매커니즘

Hybrid Search를 실행하기 전에 Metadata를 활용해 Pre-filtering 단계를 거쳐 연산의 효율성을 극대화합니다.

1. **다중 구조 기반 필터 실행 (Bitset 생성)** 
* **Exact Match (역인덱스) :** 부서명, 카테고리 등 일치 검색이 필요한 필드는 역인덱스를 통해 즉시 매칭합니다.
* **Range/Sorting/Aggregate (Doc Values) :** 날짜 범위, 가격, 버전 등 수치 비교가 필요한 필드는 Doc Values를 스캔하여 조건 부합 여부를 판단합니다.
* **Bitset 생성 :** 위 결과들을 조합하여 메모리 상에 문서 ID별로 0(제외)과 1(포함)을 표시한 **Bitset (지식 지도)** 을 형성합니다.

2. **Hybrid Search와의 물리적 결합** 
* **Vector Search (HNSW) :** 고차원 그래프 탐색 시 Bitset이 0인 노드는 유사도 계산(Distance Calculation) 대상에서 즉시 제외합니다. `이는 CPU 자원 낭비를 막는 결정적 최적화입니다.`

* **Content Search (BM25) :** 키워드 역인덱스 스캔 시에도 Bitset에 포함된 문서들만 스코어링 대상으로 한정하여 전체 검색 속도를 높입니다.

3. **성과**
* 의미적으로 유사하더라도 비즈니스 규칙(보안, 기간 등)에 어긋나는 데이터를 원천 차단하며 전체 탐색/연산을 유효 데이터군으로 집중시킵니다. 

#### 2.2 Golden Metadata 설계 원칙

RAG 답변의 품질과 신뢰도를 결정짓는 4대 핵심 Metadata 설계 전략입니다.

* **보안 식별자 (Security Identifier):** `Dept_ID`, `Access_Level` 등을 필터링하여 사용자 권한에 따른 지식 접근을 물리적으로 격리합니다.
* **시간 지표 (Temporal Index):** `Created_at` 필드를 Doc Values로 관리하여 최신 개정 규정을 우선 검색하거나 `최근 N개월` 내의 정보만 참조하도록 강제하여 정보의 시의성을 확보합니다.
* **구조적 문맥 (Structural Context):** 파편화된 Chunk가 원본의 어떤 Section_Header에서 파생되었는지 저장합니다. 이는 검색 결과 노출 시 사용자에게 원문의 위치를 안내하고 LLM이 계층적 맥락을 파악하도록 돕습니다.
* **버전 관리 (Version Control)**: `Version`, `Is_Active` 필드를 통해 동일 문서의 중복 검색을 방지하고 반드시 승인된 최신 지식(Golden Source)만이 생성 단계(Generation)로 넘어가도록 필터링합니다.

---

### 3. Hybrid Search : Lexical & Semantic Search

단일 검색 엔진은 반드시 한계를 가집니다. 키워드의 **정확성(Lexical)** 과 문맥의 **유연성(Semantic)** 을 동시에 잡기 위해 Hybrid 검색 전략을 사용합니다.

#### 3.1 BM25(Lexical Search) : 텍스트의 정밀도
* **원리 :** TF-IDF를 계승하여 단어의 빈도(TF)와 문서의 희소성(IDF)을 확률론적으로 계산합니다.
* **강점 :** 특정 고유명사, 제품 번호(예: ERP-1024), 전문 용어처럼 한 글자라도 틀리지 않아야 하는 검색에서 벡터 검색보다 압도적인 정확도를 보입니다.

#### 3.2 HNSW(Semantic Search) : 문맥의 이해
* **원리 :** 텍스트를 고차원 벡터로 변환하여 의미적 거리(Cosine Similarity)를 계산합니다.
* **강점 :** 사용자의 모호한 질문(`쉴 때 필요한 서류`)을 의미적 의도(`연차 신청서`)로 해석합니다. 단어 일치 여부를 넘어 **사용자의 검색 의도(Search Intent)** 를 파악하는 핵심 역할을 합니다.

#### 3.3 RRF (Reciprocal Rank Fusion) : 수학적 순위 통합

* 서로 다른 점수 체계(BM25의 정수형 스코어 vs 벡터의 0~1 유사도)를 결합하기 위해 순위 기반 통합 알고리즘인 RRF를 채택합니다.

* **기술적 가치 :** 점수 정규화(Normalization) 과정 없이도, 두 방식 모두에서 상위에 랭크된 문서에 가중치를 부여합니다. 이를 통해 **키워드와 의미가 모두 부합하는 '최적의 지식'** 을 선별하여 LLM에게 전달합니다.

---

### 4. Data Processing Pipeline : Knowledge Optimization

데이터가 ElasticSearch에 적재되기 전의 가공 과정은 RAG 성능의 50% 이상을 결정합니다.(문맥 보존 최우선)

#### 4.1 Markdown-Aware Splitting (Spring AI)

* 단순 글자 수 기반 분할은 정보의 단절을 초래합니다. 따라서 문서의 구조를 이해하는 `MarkdownNavigator` 전략을 사용합니다.

* **Semantic Chunking :** `#, ##` 등 마크다운 헤더를 기준으로 자름으로써 하나의 chunk가 독립적인 의미를 갖게 합니다
* **Chunk Size & Overlap :** LLM의 Context Window를 고려하여 500~1000 토큰으로 설정하되 이전 chunk의 끝부분을 다음 chunk에 **10~15% 중첩(Overlap)**시켜 정보의 연속성을 보장합니다.

#### 4.2 Context Injection & Enrichment

분할된 텍스트 조각이 파편화되지 않도록 `Metadata 강제 주입` 기법을 적용합니다.

* **Pre-pend Metadata :** 조각 본문 상단에 [문서명: 인사규정 | 위치: 제2장 휴가 > 제3절 연차]와 같은 경로 정보를 텍스트로 삽입하여 임베딩합니다.
* **기대 효과 :** 검색된 조각이 LLM에게 전달되었을 때, LLM은 해당 텍스트가 어떤 문서의 어느 맥락에서 파생되었는지 명확히 인지하여 답변의 정확도를 높입니다.

---

### 5. Sequence : RAG Workflow

사용자의 자연어 질문이 최종 답변으로 변환되는 로우레벨 프로세스는 다음과 같습니다.

#### 5.1 Request & Intent Analysis (Spring AI) 
* 사용자의 자연어 질문을 LLM이 분석하여 검색 키워드와 **Metadata 필터(부서, 날짜 등)** 를 구조화된 JSON으로 추출합니다.

#### 5.2 Query Orchestration
* Spring AI가 분석된 결과를 바탕으로 ES 전용 **Hybrid Query DSL(bool query + knn)** 을 동적으로 빌드합니다.

#### 5.3 Engine Execution (ElasticSearch)
* Pre-filtering: Metadata(역인덱스/Doc Values)로 Bitset 생성
* Concurrent Search: 생성된 Bitset 범위 내에서 BM25 스캔과 HNSW 탐색을 병렬 수행
* Re-ranking: 두 결과 세트를 RRF로 병합하여 최적의 Top-K Document 선별

#### 5.4 Context Construction
* 추출된 조각들을 프롬프트 템플릿에 주입하여 LLM을 위한 최종 컨텍스트를 구성

#### 5.5 Response Generation
* LLM은 주입된 지식 범위 내에서 사실에 근거한 답변을 생성하여 사용자에게 전달합니다

---

### 6. Expansion : Common Module(General vs RAG)
 
공용 모듈 설계를 통해 하나의 엔진 내에서 데이터 성격에 따라 두 가지 성격으로 사용이 가능하며  **"필요한 물리 구조만 선택적으로 활성화하여 리소스를 최적화"** 합니다.

#### 6.1 일반 현업 검색 (General Search)

* 정밀한 키워드 매칭과 비즈니스 정렬(최신순, 인기순)이 중요한 도메인 용도
* **메커니즘 :** Inverted Index(BM25) 매칭 + Doc Values 필터링/정렬
* **특징 :** 인덱싱 시 **HNSW 기능을 비활성화**하여 인덱스 크기를 줄이고 검색 시 벡터 연산 없이 키워드 스캔만 수행

#### 6.2 RAG 기반 검색 (Semantic RAG)

* 문맥적 의미 파악과 LLM 전달을 위한 지식 추출이 중요한 도메인 용도
* **메커니즘 :** Dense Vector(HNSW) 탐색 + Hybrid Search(RRF) 통합
* **특징 :** 인덱싱 시 `dense_vector` 필드 및 **HNSW 인덱스를 활성화** + 검색 시 질문 임베딩을 통한 유사도 탐색 수행


---


### ✅ Conclusion
* 다음 포스팅에서는 본 Architecture를 바탕으로 **Docker 기반 ElasticSearch 8.x, Spring AI**등 실제 Infra 구축 과정을 상세히 기록하고자 합니다.