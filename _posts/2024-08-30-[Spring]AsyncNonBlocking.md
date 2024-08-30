---
title: "Asynchronous Non-Blocking I/O(AIO)"
excerpt: "[Spring] About Async/sync & Blocking/NonBlocking Combination"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2024-08-30
---


#### 0. 들어가면서

  - 다음 토이 프로젝트의 Spring Web Flux를 도입해보고자 합니다. 대규모 트래픽 처리와 성능 최적화에 대해 관심이 가지고 있었고 Spring Web Flux에 대한 학습 방향을 먼저 잡았습니다. 먼저 이번 포스트를 시작으로 비동기 논블로킹 모델 및 동기/비동기, 블로킹/논블로킹 방식에 대해서 정리하고자 합니다.

  - ##### Conception

    - Blocking/Non-Blocking
      - Process의 제어권 및 흐름
    - Synchronous/Asynchronous
      - 작업 결과의 사용 여부

  - ##### Example Premise
    - A : A1, A2 특정 작업을 수행하는 Worker
    - B : A1`, B1  특정 작업을 수행하는 Worker

<br />

---

#### 1. Syncronous and Asyncronous

  - Synchronous(동기)와 Asyncronous(비동기)는 요청한 작업에 대해 완료 여부에 따라 다르게 처리되는 작업 방식을 뜻함
  - 즉, 특정 작업의 주체성을 누가 가지고 있는지
  - Request with Callback
    
  - ##### 1.1 Syncronous

    - 동기식 방식은 작업을 순차적으로 처리함
    - 작업의 주체성을 A가 가지고 있음
      
    - ###### Process
      - 1.Worker A가 A1 작업을 마친다.
      - 2.Worker A가 Worker B에게 A2 작업 전에 A` 작업을 요청하고 결과를 기다린다.
      - 3.Worker B는 A1` 작업을 마치고 Worker A에게 결과를 전달한다.
      - 4.Worker A는 A`의 결과를 받고 A2 작업을 시작한다.



  - ##### 1.2 Asyncronous

    - 비동기식 방식은 순차적인 작업 방식을 보장하지 않음
    - 작업의 주체성을 B가 가지고 있음
      
    - ###### Process
      - 1.Worker A가 A1 작업을 마친다.
      - 2.Worker A가 Worker B에게 A` 작업을 요청하고 A1 작업을 이어서 진행한다.
      - 3.Worker B에게는 두 종류의 작업 선택권이 존재한다.
        - Worker B는 A1` 작업을 마치고 Worker A에게 결과를 전달한다.
        - Worker B는 B1 작업을 마무리하고 A` 작업을 시작한다.
      - 4.Worker A는 A1 작업을 마무리하고 A2 작업을 시작한다.
    
  - ##### 1.3 Difference

    - Synchronous
      - 호출하는 함수가 해당 작업의 결과 값을 받아서다음 작업 수행
    - Asynchronous
      - 호출하는 함수가 해당 작업의 결과 값에 상관 없이 호출되는 함수가 callback으로 결과값을 넘겨 주는 방식

    - **Compare**

      ![image info](/assets/img/SyncAsync.png)
      <img src="/assets/img/SyncAsync.png" alt="" width="0" height="0">


<br />

---

#### 2. Blocking and Non-Blocking


  - Blocking과 Non-Blocking은 어떤 작업을 요청하고 결과를 기다리는지 아닌지에 따라 구분된다.
  - 즉, 작업의 흐름을 멈추는지 안멈추는지 여부에 따라 다르다.
    
  - ##### 2.1 Blocking

    - Worker A가 Worker B를 무작정 기다린다.
    - 작업의 책임을 누가 가지고 있느냐를 떠나서 작업자 A쪽의 작업을 멈춘다.
      

  - ##### 2.2 Non-Blocking

    - Worker A가 Worker B를 기다리지 않고 본인이 하던 작업 진행한다.
    - 작업의 책임을 누가 가지고 있느냐를 떠나서 작업자 A쪽의 작업은 계속 진행한다.


  - ##### 2.3 Difference

    - Blocking
      - 특정 작업에게 실행 명령과 함께 제어권을 넘겨줘서 **작업이 끝나야 제어권을 넘겨 받는다.**
    - Non-Blocking
      - 특정 작업에게 **실행 명령만 내리고 제어권은 바로 넘겨 받는다.**

    - **Compare**

      ![image info](/assets/img/BlockingNonBlocking.png)
      <img src="/assets/img/BlockingNonBlocking.png" alt="" width="0" height="0">



<br />

---

#### 3. Combination of Sync/Async and Blocking/Non-Blocking

  - Sync/Async와 Blocking/Non-Blocking은 서로 조합되어 4가지 방식이 존재
  - 자연스러운 조합도 있지만 직관적으로 부자연스러운 조합도 있음
  - Synchronous & Blocking, Asynchronous & Non-Blocking 두가지 조합은 자연스러운 조합

  - Combination
    - Sync-Blocking
    - Sync-Non-Blocking
    - Async-Blocking
    - Async-Non-Blocking

  - **Compare**

  ![image info](/assets/img/Combination.png)
  <img src="/assets/img/Combination.png" alt="" width="0" height="0">

  - ##### 3.1 Details

    - ###### Sync-Blocking
      
      - 1.Worker A가 Worker B에게 작업을 요청하고 기다린다.
      - 2.Worker B가 작업 A1`을 끝내고 결과를 Worker A에게 전달한다.
      - 3.Worker A는 Worker B에게 결과를 반환 받고 나서 나머지 작업를 진행한다.
      <br />
      - Read/Write

    - ###### Async-Non-Blocking
      - 1.Worker A가 Worker B에게 작업을 요청하고 본인의 작업으로 다시 돌아온다.
      - 2.Worker B는 Worker A에게 받은 작업을 진행하고 Worker A는 본인의 작업을 진행한다.
      - 3.Worker B의 작업이 완료되면 Worker A에게 작업 결과를 전달한다.

      <br />

      - 성능과 자원 효율적 사용 관점에서 가장 유리한 조합
      -  AIO(NIO2)

    - ###### Async-Blocking
      - 1.Worker A가 Worker B에게 작업을 요청하고 본인의 작업으로 다시 돌아온다.
      - 2.Worker B는 Worker A에게 받은 작업을 진행한다.
      - 3.Worker A는 중간에 Worker B에게 요청한 작업에 대한 중간 결과를 보고 받아서 처리해야 한다.(Worker A는 Worker B에게 요청을 해서 중간 결과를 기다린다.)
      - 4.요청의 결과를 받은 Worker A는 결과를 통해 자신의 작업을 처리하고 Worker B는 Worker A에게 받은 작업을 이어서 처리한다.

      <br />

      - 3, 4번 작업은 Worker A가 Worker B에게 맡긴 작업이 완전히 종료되기 전까지 반복된다.
      - I/O multiplexing(select/poll)

    - ###### Sync-Non-Blocking
      - 1.Worker A가 Worker B에게 작업을 요청한다.
      - 2.Worker B는 Worker A에게 받은 작업을 진행하고 Worker A는 작업을 하지 않고 Worker B가 하는 일을 확인만 한다.
      - 3.Worker A는 중간에 Worker B에게 요청한 작업에 대한 중간 결과를 확인만 한다.(Worker A는 Worker B에게 요청한 중간 결과를 기다리지 않는다.)
      - 4.Worker B의 작업이 모두 마무리 됬을 때 Worker A는 본인의 작업을 진행한다.

      <br />

      - 3번 작업은 Worker A가 Worker B에게 맡긴 작업이 완전히 종료되기 전까지 반복된다.
      - Read/Write(O_NONBLOCK)

  - ##### 3.2 Java AIO(NIO2)

    - 기존 Java NIO, Select, Epoll 사용할 경우
      - Kunnel에서 Application으로 준비 완료 이벤트(read, write..)를 전달해주지 않고 Application에서 Kunnel로 직접 주기적 확인이 필요했다.(Sync Non-Blocking)
      - Busy-Wait 발생
    - Java AIO(NIO2) 사용할 경우
      - Kunnel은 I/O 작업을 비동기적으로 처리한다.
      - 작업이 완료되면 Kunnel이 감지하여 Application으로 준비 완료 이벤트(read, write..)를 알린다.
      - Non-blocking
        - Return 값으로 Future이나 Method Parameter로 callback(CompletionHandler)를 등록하낟.
        - Response with Callback
      - Asynchronous
        - 준비 완료 이벤트를 Kunnel이 Application(AIO 객체)으로 알리고 해당 Application은 등록된 callback을 실행 또는 리턴(Future)의 상태를 업데이트한다.
<br />


---

### Reference


- https://inpa.tistory.com/entry/%F0%9F%91%A9%E2%80%8D%F0%9F%92%BB-%EB%8F%99%EA%B8%B0%EB%B9%84%EB%8F%99%EA%B8%B0-%EB%B8%94%EB%A1%9C%ED%82%B9%EB%85%BC%EB%B8%94%EB%A1%9C%ED%82%B9-%EA%B0%9C%EB%85%90-%EC%A0%95%EB%A6%AC
- https://velog.io/@kangdev/%EA%B8%B0%EC%88%A0%EB%A9%B4%EC%A0%91JS
- https://www.inflearn.com/news/72620?gad_source=1&gclid=CjwKCAjwuMC2BhA7EiwAmJKRrBpR65Tof5nIrNHqNbYxqskFMjkwUa92A6wyx8XNyM6Yz6PbrFMuIhoCJ4IQAvD_BwE
- https://jammdev.tistory.com/168
- https://etloveguitar.tistory.com/140
- https://choi-geonu.medium.com/%EB%B0%B1%EC%97%94%EB%93%9C-%EA%B0%9C%EB%B0%9C%EC%9E%90%EB%93%A4%EC%9D%B4-%EC%95%8C%EC%95%84%EC%95%BC%ED%95%A0-%EB%8F%99%EC%8B%9C%EC%84%B1-2-%EB%B8%94%EB%A1%9C%ED%82%B9%EA%B3%BC-%EB%85%BC%EB%B8%94%EB%A1%9C%ED%82%B9-%EB%8F%99%EA%B8%B0%EC%99%80-%EB%B9%84%EB%8F%99%EA%B8%B0-e11b3d01fdf8
- https://dokit.tistory.com/35
- https://starryeye.tistory.com/205
- https://homoefficio.github.io/2017/02/19/Blocking-NonBlocking-Synchronous-Asynchronous/