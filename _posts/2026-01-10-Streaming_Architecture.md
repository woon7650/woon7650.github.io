---
title: "Scalable Data Streaming Optimization: The Limits of Contiguous Memory and JVM Resource Governance"
excerpt: "[Project Experience] 양방향 Streaming 설계부터 G1GC Humongous 영역 분석까지"
date: 2026-01-10
categories: [Java, Spring, Infrastructure]
tags: [RestTemplate, HttpClient, NIO, DirectBuffer, JVM, G1GC, ZeroCopy, MemoryManagement]
---


### 들어가면서
대규모 데이터 처리에서 성능만큼 중요한 것은 **Resource Governance**입니다. 특히 하루 1~2회 실행되는 배치 시스템은 상시 가동되는 API 서버와 달리 **자원의 집중 투입과 즉각적인 회수**가 핵심입니다. 작업 종료 후에도 리소스를 점유하는 것은 시스템 전체의 낭비이자 운영 리스크이기 때문입니다.

본 포스팅에서는 RestTemplate의 Buffering 방식이 가진 '연속된 메모리 할당'의 한계를 분석하고 양방향 스트리밍 도입을 통한 개선 성과를 공유합니다.

---

### 1. Buffering의 한계: 왜 '연속된 Memory 공간'이 문제일까?

우리가 흔히 사용하는 `RestTemplate`의 기본 방식인 **Buffering**이 대용량 데이터 전송에서 실패하는 근본 원인은 단순히 Memory 총량이 부족해서가 아닙니다. 자바 Memory 할당의 핵심 원칙인 **'연속된 Memory 공간(Contiguous Memory Space)'** 확보에 실패하기 때문입니다.

자바의 배열(`byte[]`)이나 큰 객체 덩어리는 Memory상에서 파편화되지 않은 하나의 거대한 연속된 공간을 점유해야 합니다. 만약 500MB 파일을 전송하려 할 때 전체 Heap 여유 공간이 1GB가 있더라도 Memory 여기저기에 작은 객체들이 박혀 있어 **연속된 500MB 공간**을 찾을 수 없다면 서버는 즉시 `OutOfMemoryError(OOM)`를 던지며 멈춥니다.

**Streaming 방식**은 이 거대한 덩어리를 8KB라는 작은 조각(Chunk)으로 쪼개어 전송합니다. 이는 연속된 Memory 확보에 대한 물리적 부담을 완전히 제거하여 아무리 큰 데이터라도 일정한 Memory 안에서 안전하게 보낼 수 있게 해줍니다.

---

### 2. 양방향 파이프라인: Request와 Response를 동시에 흐르게 하기

진정한 최적화는 '보내는 과정'뿐만 아니라 '받는 과정'까지 모두 Streaming되어야 완성됩니다. 서버는 데이터를 담아두는 창고가 아니라 데이터가 흐르는 **통로(Pipeline)**가 되어야 합니다.

#### 1) Request Streaming (Outbound)
`setBufferRequestBody(false)` 설정을 통해 Engine은 본문을 Memory에 쌓지 않고 실시간으로 선로에 밀어냅니다. 파일의 위치 정보만 들고 있다가 Network가 비는 시점에 8KB씩 퍼 올리는 '지도 기반 전송' 구조를 가집니다.

#### 2) Response Streaming (Inbound)
응답을 받을 때도 일반적인 메서드는 응답 전체를 다시 Heap에 담으려 합니다. 대신 `RestTemplate.execute()`와 `ResponseExtractor`를 사용하면 서버로부터 오는 응답 스트림(`InputStream`)을 우리 서버의 파일 시스템이나 출력 스트림(`OutputStream`)에 직접 꽂아 넣을 수 있습니다.

---

### 3. Network Engine의 규칙: Apache HttpClient의 상태 머신

Streaming 모드에서 하부의 **Apache HttpClient** Engine은 매우 엄격한 **전송 순서(State Machine)**를 따릅니다.

1.  **Header 준비 :** 목적지 주소와 콘텐츠 타입을 먼저 정의합니다.
2.  **Engine 승인 :** Engine에게 전송 준비가 됐음을 알리는 신호를 보냅니다.
3.  **Header 송출 :** Engine이 선로를 열고 헤더 패킷을 먼저 보냅니다.
4.  **Body 개방 :** 헤더가 성공적으로 나간 뒤에야 실제 데이터를 쓸 수 있는 통로(`getBody()`)가 열립니다.

이 순서를 무시하고 바로 데이터를 쓰려고 하면 Engine은 예외를 발생시킵니다. 이때 `HttpMessageConverter`를 사용하면 컨버터가 Engine과 대화하며 이 복잡한 순서를 대신 맞춰주는 **Orchestrator** 역할을 수행합니다.

---

### 4. G1GC 내부 동작: Humongous 할당과 초기 GC 부하

배치 작업 초기에 GC 지표가 급증하는 현상은 G1GC의 **Region** 관리 전략과 밀접한 관련이 있습니다.

* **Humongous Object:** G1GC는 Heap을 일정한 크기의 구역(Region)으로 나눕니다. 객체 크기가 구역 절반보다 크면 '거대 객체(Humongous)'로 보고 특별 취급을 합니다. Buffering 방식을 쓰면 전송 데이터 자체가 거대 객체가 되어 Memory 효율을 극도로 떨어뜨립니다.
* **초기 GC 부하:** Streaming 방식을 써도 초기에 100+개의 Connection을 동시에 맺거나 라이브러리를 로딩할 때 구역들을 급격히 차지하면 JVM은 공간 확보를 위해 빈번한 GC를 수행합니다. 이는 Memory 부족이 아니라 대형 손님을 맞이하기 위한 **JVM의 인프라 정비 작업**입니다.

---
### 5. 효율의 증거: Allocated/Promoted Rate 지표 읽기

설계가 얼마나 효율적인지는 Heap 안에서 객체가 어떻게 이동하는지를 보면 증명됩니다.

* **Allocated Rate (할당률):** 초당 Heap에 생성되는 객체 양입니다. Streaming 적용 시 8GB 데이터를 처리해도 할당률이 매우 일정하게 유지됩니다. 8KB 버퍼 하나를 계속 재사용(**Buffer Pool Reuse**)하기 때문입니다.
* **Promoted Rate (승급률):** Young 영역에서 살아남아 Old 영역으로 넘어가는 객체 양입니다. 효율적인 Streaming은 객체를 쓰고 바로 버리기 때문에 승급률을 거의 제로에 가깝게 만듭니다. 


* **Buffering -> Streaming 변경 후에 실무에서 지표상의 낮은 승급률(785KB/s)을 확인했고 시스템이 장기적으로도 매우 건강하다는 부분을 확인할 수 있었습니다.**

---

### 6. Native Memory의 비밀: 8KB 버퍼가 4GB 점유가 되는 이유

NIO의 핵심인 **Direct Buffer**는 커널과 유저 영역 사이의 복사 과정을 생략하는 **Zero-copy**를 지향합니다. 하지만 이 과정에서 Memory 점유 현상이 발생합니다.

1.  **누적된 조각들:** 8,000여 건의 요청 동안 매번 새로운 `DirectByteBuffer`(Proxy 객체)가 생성되어 네이티브 Memory 영역에 층층이 쌓입니다.
2.  **Memory 파편화:** OS는 Memory를 딱 8KB씩 빌려주지 않고 페이지 단위로 넉넉하게 빌려주다 보니 실제 사용량보다 더 많은 공간이 예약됩니다.
3.  **리소스 유보(Retention):** 이는 Memory 누수가 아니라 다음에 또 쓸 것을 대비해 JVM이 OS로부터 빌려온 리소스를 쥐고 있는 상태입니다.

---

### 7. Native Memory Retention: 왜 자원 반납이 지연되는가?

작업이 끝났음에도 Native Memory가 해제되지 않고 유지되는 현상은 JVM의 **Cleaner** 구조와 **지연 해제(Deferred Deallocation)** 특성 때문입니다. 단순히 메모리 누수가 아니라 자원 회수 메커니즘이 동작할 '조건'이 충족되지 않은 상태로 이해해야 합니다.

* **Cleaner와 연결 고리:** Native Memory는 Heap 영역에 있는 아주 작은 **Proxy 객체(DirectByteBuffer)** 가 GC에 의해 수집될 때 비로소 해제됩니다. 이때 `Cleaner(PhantomReference)` 스레드가 작동하여 실제 메모리 주소를 OS에 반납합니다.
* **역설적인 상황(지연 해제):** 앞서 언급한 스트리밍 최적화로 인해 Heap 메모리 상태가 매우 안정적이면 JVM은 GC를 수행할 필요가 없다고 판단합니다. 결국 Heap에 있는 Proxy 객체들이 정리되지 않고 남게 되며 결과적으로 Native Memory 청소 기회까지 사라지는 리소스 고착 현상이 발생합니다.
---

### 8. 실무적 해제 전략: 기술 스택 및 환경별 리소스 거버넌스

리소스 반납의 책임을 JVM의 자율에만 맡기지 않고 엔지니어가 직접 제어하기 위한 기술 스택별 최적의 전략은 다음과 같습니다.

#### 전략 1. RestTemplate 환경: 명시적 자원 반납
`RestTemplate`은 자바 표준 `DirectByteBuffer`를 사용하므로 JVM의 GC 사이클을 활용하는 것이 가장 안전하고 보편적입니다.
* **의도적 GC 유도:** 대용량 데이터 전송 배치가 완전히 종료되는 시점에 `System.gc()`를 호출합니다. 이는 메모리 부족 때문이 아니라, **Cleaner를 강제로 기동**시켜 점유된 네이티브 자원을 OS에 즉시 반납하기 위한 전략적 선택입니다.
* **운영 포인트:** 데이터 전송 로직이 끝난 직후 명시적으로 호출하여 Native Memory 점유 시간(Retention Time)을 최소화합니다.

#### 전략 2. JVM 및 인프라 레벨: 상한선 제어 전략
코드를 수정하기 어렵거나 인프라 차원의 자동화된 안전장치가 필요할 때 사용하는 거버넌스 전략입니다.
* **`-XX:MaxDirectMemorySize` 설정:** Native Memory의 상한선을 물리 메모리 상황에 맞춰 타이트하게 지정합니다. 점유량이 이 임계치에 도달하면 JVM은 **스스로 Full GC를 트리거**하여 `Cleaner`를 작동시킵니다.
* **트레이드 오프:** 너무 낮게 설정하면 빈번한 GC로 애플리케이션 성능이 저하될 수 있으므로 예상 피크치 대비 20~30%의 여유를 두는 것이 좋습니다.

#### 전략 3. WebClient/Netty 환경: 직접 수명주기 관리 (Reference Counting)
비동기 Non-blocking 기반 스택에서는 GC에 의존하지 않는 독자적인 메모리 관리 체계를 사용합니다.
* **Reference Counting:** Netty의 `ByteBuf`는 `retain()`과 `release()`를 통해 참조 횟수를 직접 관리합니다. 사용 즉시 `release()`를 호출하면 GC를 거치지 않고 **메모리 풀(Pool)로 즉시 반납**되거나 시스템에 반환됩니다.
* **Reflection 기반 강제 해제:** 극단적인 저지연이 필요한 경우 `DirectBuffer`의 내부 `cleaner().clean()`을 리플렉션으로 직접 호출하여 즉시 파괴합니다. 단 해제된 버퍼를 다시 참조할 경우 JVM 크래시(Segmentation Fault) 위험이 있어 매우 정교한 생명주기 설계가 뒷받침되어야 합니다.

#### 사용 전략
* 본 프로젝트에서는 일 배치 후 **HttpClient Connection을 명시적으로 반납**합니다. 이 경우 커넥션에 할당되었던 Direct Buffer들이 자연스럽게 해제 대상이 됩니다.
---

### 9. ✅ Conclusion

이번 프로젝트를 통해 8KB 단위로 흐르는 데이터가 어떻게 수 GB의 시스템 리소스와 상호작용하는지 깊이 있게 이해할 수 있었습니다.

1.  **연속된 Memory의 탈피:** 대용량 데이터는 조각내어 흐르게 할 때 가장 안전합니다.
2.  **양방향 파이프라인:** 보내고 받는 모든 과정을 스트림으로 연결해 Memory 점유를 제어했습니다.
3.  **지표 기반 거버넌스:** Humongous 할당과 Cleaner 원리를 이해하여 JVM의 동작을 예측하고 통제할 수 있게 되었습니다.
