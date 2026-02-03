---
title: "Setting Hybrid RAG Architecture : Gradle & Spring AI Milestone"
excerpt: "[Java] Gradle 의존성과 Spring AI Milestone 활용 가이드"
date: "2026-02-03"
categories: [Backend, Infrastructure, AI]
tags: [SpringAI, Gradle, Milestone, ElasticSearch, DependencyManagement, BOM, JDK21]
---

### 0. 들어가면서 : 기술의 속도와 프레임워크의 격차
현대 AI 생태계 특히 RAG(Retrieval-Augmented Generation) 분야의 변화 속도는 프레임워크의 정식 릴리스 주기를 상회합니다. **Python 진영의 LangChain, LangGraph**가 독주하는 상황에서 **Java 진영의 Spring AI** 역시 그 간극을 메우기 위해 급진적인 발전을 거듭하고 있습니다.

ElasticSearch의 Hybrid Search 기능을 프로토타이핑하기 위해 우리는 아직 정식 버전이 아닌 **Spring AI Milestone(M5)** 버전을 선택했습니다. 단순히 version 숫자를 기입하는 수준을 넘어 Gradle이 원격 저장소에서 어떤 메커니즘으로 라이브러리를 탐색하고 내 프로젝트로 추상화하여 가져오는지에 대해 정리해보고자 합니다.

---

### 1. Artifact Repository


#### 1.1 Milestone 저장소의 GAV 체계
Gradle은 GAV(Group, Artifact, Version)를 기반으로 URL을 조합하여 필요한 자원을 탐색합니다.
* **Group (`org.springframework.ai`)** : 저장소 내의 폴더 경로로 치환됩니다. (`org/springframework/ai`)
* **Artifact (`spring-ai-elasticsearch-store`)** : 해당 기능이 담긴 모듈 폴더입니다.
* **Version (`1.0.0-M5`)** : 특정 시점에 동결된 배포본 폴더입니다.

#### 1.2 JAR vs Sources-JAR : 완성품과 설계도
저장소 내부에는 실행 파일뿐만 아니라 개발을 돕는 다양한 형태의 파일이 공존합니다.
* **`.jar` (Binary)** : 컴파일된 바이트코드(`.class`)의 집합이며 JVM이 실제로 실행하는 **완성품**입니다.
* **`-sources.jar` (Source)** : 사람이 읽을 수 있는 **자바 소스 코드(`.java`)** 입니다. IDE에서 내부 로직을 클릭했을 때 설계도를 보여주는 역할을 합니다.
* **`.pom` (Metadata)** : 이 부품을 조립하기 위해 필요한 `Transitive Dependencies` 리스트가 담긴 명세서입니다.

---

### 2. Dependency Management

과거에는 필요한 JAR를 일일이 다운로드하여 `libs` 폴더에 넣었지만 현대적 빌드 도구는 이 과정을 자동화하여 '의존성 지옥'을 해결합니다.

#### 2.1 BOM (Bill of Materials) : 버전의 오케스트레이션
Spring AI처럼 모듈이 수십 개인 프로젝트는 버전 충돌이 치명적입니다. 
* **매커니즘**: `spring-ai-bom`을 도입하면 개별 모듈의 버전을 일일이 적지 않아도 됩니다. BOM이 `M5 세트`에 맞는 최적의 버전 조합을 자동으로 결정합니다.
* **가치**: 라이브러리 간의 함수 호출 불일치(`NoSuchMethodError`)를 원천 차단하여 런타임 안정성을 확보합니다.

#### 2.2 Resolution & Caching
1. **조회(Resolution)**: `repositories`에 등록된 순서대로 원격 저장소를 탐색합니다.
2. **다운로드**: 발견된 부품을 로컬 캐시(`~/.gradle/caches`)에 저장하여 다음 빌드 속도를 비약적으로 높입니다.
3. **연결**: 내 프로젝트의 클래스패스(Classpath)에 해당 JAR를 주입하여 즉시 컴파일 가능하게 만듭니다.

---

### 3. Implementation : Example of `build.gradle`

```java
repositories {
    mavenCentral()
    // Spring AI Milestone 전용 저장소
    maven { url 'https://repo.spring.io/milestone' } 
}


dependencyManagement {
    imports {
        // 모든 Spring AI 관련 라이브러리의 버전을 1.0.0-M5로 일괄 동기화
        mavenBom "org.springframework.ai:spring-ai-bom:1.0.0-M5"
    }
}

dependencies {

    implementation 'org.springframework.ai:spring-ai-elasticsearch-store-spring-boot-starter'
    
    implementation 'org.springframework.ai:spring-ai-openai-spring-boot-starter'
    
    implementation 'org.springframework.boot:spring-boot-starter-web'
    
}

```

---