---
title: "Java’s New Concurrency Paradigm: Virtual Thread Deep Dive(Part 2)"
excerpt: "[Java] Virtual Thread 이해 및 활용을 위한 이론 정리"
date: 2026-01-13"
categories: [Java, Backend, Infrastructure]
tags: [VirtualThread, ForkJoinPool, WorkStealing, Continuation, MemoryManagement, BackendArchitecture]
---

#### 들어가면서
Part 1에서는 Virtual Thread의 기본 개념과 Platform Thread와의 차이점을 살펴보았습니다. Part 2에서는 **JVM이 어떻게 Thread의 주도권을 OS로부터 가져왔는지**와 그 이면에서 동작하는 **Continuation 객체 처리와 Scheduling 알고리즘**을 정리해보고자 합니다.

---

#### 1. Thread 자주권의 확보: OS Kunnel에서 JVM 유저 영역으로

Virtual Thread의 본질은 **Thread Scheduling 주체의 완벽한 이전**에 있습니다.

* **Kunnel 모드에서 유저 모드로:** 과거의 자바 Thread는 OS Kunnel Thread의 대리인이었습니다. Thread를 멈추고 실행할 때마다 고비용의 **Context Switching**이 발생했습니다.
* **JVM의 독립:** Virtual Thread 시대의 JVM은 내부에 **전용 ForkJoinPool Scheduler**를 직접 운영합니다. 이제 어떤 Virtual Thread를 실행하고 멈출지 OS의 허락을 받지 않고 JVM 내부에서 스스로 결정합니다.
* **물리적 차단에서 논리적 대기로:** Kunnel이 Thread를 중단시키는 '물리적 차단' 대신 JVM이 Virtual Thread 객체를 Heap에 잠시 주차시키는 '논리적 대기'로 전환되었습니다. 이를 통해 시스템은 수십만 개의 Thread를 관리하면서도 Kunnel 부하를 최소화합니다.

---

#### 2. 핵심 매커니즘: Continuation과 Stack 프레임의 이동

Virtual Thread가 Carrier Thread에서 내려오고 다시 올라오는 과정은 **데이터 복사**의 과정입니다.

1. **Unmount (Yield):** Virtual Thread가 Blocking I/O를 만나면 현재 실행 중인 CPU의 레지스터 상태와 스택 프레임(Local Variables, Return Address 등)을 `Continuation`이라는 객체에 담아 Heap Memory로 복사**합니다.
2. **Mount (Resume):** I/O가 완료되어 다시 차례가 오면 Heap에 저장되어 있던 `Continuation` 데이터를 다시 Carrier Thread의 물리 스택으로 복사해 옵니다.
3. **트레이드오프:** 이 과정은 Kunnel Context Switch보다는 훨씬 저렴하지만 빈번한 발생 시 **Memory I/O 비용과 GC 부하**를 발생시킵니다. 즉 CPU를 아끼기 위해 Memory 성능을 전략적으로 지불하는 것입니다.
---

#### 3. 전용 ForkJoinPool과 이중 큐(Queue) 구조

Virtual Thread는 일반적인 `ThreadPoolExecutor`와는 완전히 다른 **Work-Stealing Architecture**를 가집니다.

* **Local Queue의 독립성:** 각 Carrier Thread는 개별 Local Queue를 가집니다. 대부분의 작업을 자기 큐에서 처리하므로 중앙 큐를 조회할 때 발생하는 **Lock Contention**이 사라져 Scheduling 병목을 해결합니다.
* **Work-Stealing 알고리즘:** 특정 Carrier Thread가 한가해지면 다른 바쁜 Carrier의 Local Queue '뒷부분'에서 작업을 몰래 가져옵니다. 
* **왜 Virtual Thread에 최적인가?:** Virtual Thread는 생성과 소멸이 매우 잦습니다. 하나의 중앙 큐만 있다면 큐 자체가 거대한 병목이 되었겠지만 분산 큐 구조를 통해 수백만 개의 작업을 효율적으로 분배합니다.

---

#### 4. Semaphore를 통한 '내부 DDoS' 방어

Virtual Thread는 상한선이 없는 **Unbounded Resource**입니다. 이는 시스템에 자연스럽게 존재하던 **Backpressure**가 사라졌음을 의미합니다.

* **위험 시나리오:** Platform Thread를 사용할 경우 Thread 풀(예: 200개)이 곧 DB 커넥션 최대치를 방어해 줬습니다. 하지만 Virtual Thread 환경에서는 10만 개의 Thread가 동시에 `hikaricp.getConnection()`을 호출할 수 있으며 이는 DB 서버를 즉시 마비시키는 '내부 DDoS'가 됩니다.
* **해결책:** 이제 Thread Pool Size가 아닌 **Semaphore**를 사용하여 공유 Resource에 대한 **논리적 진입 장벽**을 직접 설계해야 합니다. 
* **효율적 대기:** 10만 개 중 99990개가 세마포어 대기 큐(`AQS` 기반)에서 대기하더라도 이들은 CPU를 쓰지 않는 'Heap 위의 객체'일 뿐이므로 Memory 소모 외에 시스템에 큰 부하를 주지 않습니다.

---
#### 5. Pinning과 CPU Bound

Virtual Thread 효율을 망치는 가장 큰 적은 **Pinning** 현상입니다. Virtual Thread가 Carrier Thread에서 내려오고 싶어도 내려오지 못하는 상황입니다.

1. **synchronized의 기술적 한계:** `synchronized`는 자바 초기에 설계된 OS Monitor Lock을 사용합니다. 이 안에서 I/O가 발생하면 JVM은 스택 프레임을 안전하게 추출할 수 없습니다. 따라서 Virtual Thread가 Carrier에 묶여버리고 다른 Virtual Thread들은 실행 기회를 잃습니다.
2. **CPU Bound의 함정:** Virtual Thread는 '대기'할 때 자리를 비워주는 모델입니다. `parallelStream()`이나 암호화 연산처럼 CPU를 계속 쓰는 작업은 '대기'가 아니므로 자리를 비우지 않습니다. 이런 작업은 여전히 **Platform Thread Pool**이 훨씬 효율적입니다.

---

#### ✅ Conclusion

Virtual Thread는 Thread를 아끼는 기술이 아니라 **Carrier Thread를 1초도 쉬지 않게 만드는 기술**입니다. 

1. **주도권:** Thread Scheduling은 이제 애플리케이션(JVM)의 통제 영역에 들어왔습니다.
2. **Memory와 CPU의 균형:** CPU Context Switching 비용을 줄인 대가로 Heap Memory 점유와 GC 부하가 증가함을 인지하고 관리해야 합니다.
3. **Resource 통제:** 무제한 동시성 속에서 **Semaphore와 Lock**을 통해 Resource의 임계점을 관리하는 설계 과정이 꼭 필요합니다.

