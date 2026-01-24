---
title: "Java Virtual Thread Deep Dive(Part 1)"
excerpt: "[Java] Virtual Thread 이해 및 활용을 위한 이론 정리"
date: 2026-01-12
categories: [Java, Backend, Infrastructure]
tags: [VirtualThread, ProjectLoom, JVM, Concurrency, ForkJoinPool, PerformanceTuning]
---

### 들어가면서
이번 포스트에서는 IO/NIO 최적화를 넘어 Thread 모델 자체의 한계를 극복하기 위해 진행했던 Virtual Thread를 학습한 내용에 대해서 정리해보고자 합니다. 

---

### 1. Thread 모델의 진화: Platform vs Virtual

기존 자바의 성능 병목은 **Thread-per-request** 모델에서 발생했습니다. 

* **Platform Thread:** OS 커널 Thread와 **1:1로 매핑**됩니다. 생성 시 설정에 따라 Stack Memory를 점유하며 Context Switching시 커널 모드 전환 비용이 발생합니다. 하드웨어의 한계로 인해 수천 개 이상의 Thread를 유지하기 어렵습니다.
* **Virtual Thread:** JVM이 관리하는 **경량 Thread**입니다. OS Thread와 **M:N 매핑** 방식으로 동작하며 수 KB의 최소 Memory만 사용합니다. **Thread는 Pooling해야 한다**는 고정관념을 깨고 필요할 때마다 만들고 버리는 객체 지향적 실행 단위입니다.

---

### 2. Virtual Thread의 핵심 아키텍처: Carrier와 Continuation

Virtual Thread가 물리적 한계를 극복하는 핵심은 **Carrier Thread**와 **Continuation** 기술의 조합에 있습니다.



#### 1) Carrier Thread: 실제 일을 하는 일꾼
Virtual Thread는 스스로 CPU 위에서 실행될 수 없습니다. 실제 실행을 담당하는 Platform Thread를 **Carrier Thread**라고 부릅니다. 이는 전용 Scheduler인 `ForkJoinPool`에 의해 관리되며 기본적으로 CPU 코어 개수만큼 생성되어 상시 대기합니다.

#### 2) Continuation: 실행 상태의 객체화
Virtual Thread는 실행 중 I/O(DB 호출, API 통신 등)를 만나면 현재의 실행 상태(Stack Frame)를 **Heap Memory**로 대피(**Unmount**)시킵니다. 이때 사용되던 Carrier Thread는 즉시 다른 Virtual Thread를 처리합니다. I/O가 완료되면 Heap에 있던 상태를 다시 Carrier 위로 복구(**Remount**)하여 중단된 지점부터 실행을 재개합니다.

---

### 3. 내부 Scheduling: ForkJoinPool과 Work-Stealing

Virtual Thread의 Scheduler는 효율성을 극대화하기 위해 독특한 Queue 구조를 가집니다.

* **글로벌 큐(Global Queue):** 새로 생성된 Virtual Thread 작업들이 처음 대기하는 곳입니다.
* **로컬 큐(Local Queue):** 각 Carrier Thread가 개별적으로 가진 작업 바구니입니다. 중앙 큐에서의 경합(Lock Contention)을 줄여 초고속 Scheduling을 가능하게 합니다.
* **Work-Stealing:** 특정 Carrier Thread가 한가해지면 다른 바쁜 일꾼의 로컬 큐에서 작업을 훔쳐와 CPU 활용률을 100%로 유지합니다.

---

### 4. 실무 적용 시나리오: Spring Boot와 Virtual Thread

Spring Boot 3.2+ 환경에서는 설정 한 줄로 톰캣의 워커 Thread를 Virtual Thread로 전환할 수 있습니다.
`spring.threads.virtual.enabled=true`

이 설정을 켜면 톰캣은 고정된 크기의 Platform Thread Pool 대신 요청마다 새로운 Virtual Thread를 즉석에서 생성합니다. 이는 **Concurrency**을 무제한에 가깝게 확장하여 수만 개의 동시 접속자가 DB 응답을 기다리는 상황에서도 서버가 멈추지 않게 합니다.

---

### 5. 주의사항: Yield & Pinning

Virtual Thread가 모든 차단 상황을 해결해 주지는 않습니다. 개발자는 **Pinning(고정)** 현상을 항상 경계해야 합니다.

* **착한 차단 (Yield 가능):** Virtual Thread가 똑똑하게 자리를 비워줍니다.
    * 자바 표준 I/O, `ReentrantLock`, `Semaphore` 등
* **나쁜 차단 (Pinning 발생):** 
    * **synchronized :** 내부에서 I/O 발생 시 Virtual Thread가 Carrier에 고정되어 다른 Thread가 실행되지 못합니다.
    * **JNI(Native Method) 호출:** Native Memory는 JVM이 제어할 수 없어 자리를 비워줄 수 없습니다.
    * **CPU Bound 작업:** parallelStream()이나 복잡한 연산은 **기다림이 아닌 실행 중**이므로 일꾼을 해방시키지 않습니다.

---

### 6. 리소스 거버넌스: Semaphore의 필연성

Virtual Thread 사용 시에 가장 유의해야할 점은 **무제한의 동시성**입니다. 10만 개의 요청이 들어와 10만 개의 Virtual Thread가 동시에 DB 커넥션을 요청하면 하위 시스템은 즉시 마비됩니다.

* **해결책:** 기존의 Thread Pool이 담당하던 '상한선' 역할을 **Semaphore**가 대신해야 합니다.
* **동작 원리:** 10만 개의 Virtual Thread가 생겨도 DB 접근 통로는 세마포어로 10개만 열어둡니다. 나머지 99,990개는 Heap Memory의 세마포어 대기 큐에서 아주 적은 비용으로 대기합니다. 이는 Platform Thread 10만 개가 대기할 때 필요한 수십 GB의 Memory 낭비를 막아줍니다.

---

### 7. 심화 분석: GC와 Memory Impact

Virtual Thread도 결국 Heap에 존재하는 객체입니다.

* **GC 부하:** Virtual Thread가 많아지면 GC가 훑어야 할 객체(Live Objects)가 늘어납니다. 특히 I/O 대기 중인 Virtual Thread들의 스택 정보가 Heap에 머물며 **Young/Minor GC** 빈도를 높일 수 있습니다.
* **Connection Starvation:** Carrier Thread은 적은데 너무 많은 Virtual Thread가 커넥션을 붙잡고 있으면 대기 시간이 무한정 길어질 수 있으므로 타이트한 타임아웃 설계가 병행되어야 합니다.

---

### ✅ Conclusion

Virtual Thread는 Java의 Thread를 OS의 제약으로부터 해방시킵니다. 하지만 이는 **Heap과 CPU 사이의 정교한 트레이드오프**입니다.
* Scheduling 주체 : OS -> JVM
* Memory 주체: OS Stack → JVM Heap

1. Network, DB I/O가 많은 서비스에서 최고의 성능을 발휘합니다.
2. `Pooling`이 아닌 Virtual Thread는 쓰고 버리는 소모품입니다.
3. `Semaphore`와 `ReentrantLock`을 통해 하위 리소스를 보호하고 Pinning을 방지해야 합니다.

Virtual Thread는 Thread 수를 무한정 늘리는 것이 아닌 I/O 대기를 극한으로 줄여 **시스템 전체의 처리량(Throughput)을 극대화하는 모델**입니다.