---
title:  "Process & Thread"
excerpt: "[Spring] About Spring Basics Part 3"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2023-08-10

---

### 0. 들어가면서

- Spring을 사용한다면 필수적으로 등장하는 Thread와 Process의 기본적인 개념에 대해서 다시 한 번 정리하고자함


#### Program

> 파일이 저장 장치에 저장되어 있지만 메모리에는 올라가 있지 않은 정적인 상태

<br />

---

### 1. Process

> 운영체제로부터 자원을 할당받은 작업의 단위

<br />

##### 1.1 What is Process ?

- 운영체제가 메모리 등의 필요한 자원을 할당해준 실행 중인 Program
- Program의 Instance
- 운영체제로부터 각각 독립된 memory 영역(code, data, stack, heap)을 할당 받음
- Process는 하나 이상의 Thread를 가짐
- 기본적으로 하나의 Process 생성시 하나의 Thread가 같이 생성됨(Main Thread)
- 각각의 Process는 서로의 memory에 접근할 수 없음(IPC를 이용하여 접근 가능)

<br />

##### 1.2 Structure(memory)

![image info](/assets/img/process.jpg)
<img src="/assets/img/process.jpg" alt="" width="0" height="0">

- stack : 지역, 매개 변수 같은 임시적인 데이터가 담기는 영역
- heap : Runtime때 동적으로 memory를 할당받는 영역
- data : 전역, 정적 변수가 담기는 영역
- text/code : 실행할 Program 코드
 
<br />

##### 1.3 MultiProcess

> 하나의 Program에서 여러 개의 Process로 구성하여 각 Process가 하나의 작업을 처리하도록 하는 방식

- 장점
  - 자식 Process에서 문제 발생시 자식 Process만 kill됨
- 단점
  - Context Switching에서의 오버헤드가 발생할 수 있음
    - CPU에서 여러 Process를 돌아가면서 작업을 처리하는 과정
    - Process는 각각의 독립된 memory 영역을 할당 받음
    - 따라서 매번 Cache에 있는 모든 데이터를 모두 리셋하고 다시 캐쉬정보를 불러와야함

<br />

---

### 2. Thread

> 프로세스가 할당받은 자원을 이용하는 실행 흐름의 단위

<br />

##### 2.1 What is Thread ?

- Process 내에서 Process의 자원을 이용하여 실제로 작업을 수행
- 동일한 Process 내에서 code, data, heap 영역을 공유함
- 각각의 Thread는 독자적인 stack memory를 갖음
- 한 Thread의 자원을 변경하면 다른 이웃 Thread도 변경됨

<br />

##### 2.2 Single Threaded Process & MultiThreaded Process Structure(memory)

![image info](/assets/img/thread.jpg)
<img src="/assets/img/thread.jpg" alt="" width="0" height="0">

- 동일 Prcocess 내의 Thread들은 code, data, files를 공유함
- Thread는 각각의 registers, stack을 가짐

<br />

##### 2.3 MultiTherad

> 하나의 Program을 여러 개의 Thread로 구성하고 각 Thread로 하나의 작업을 처리하도록 하는 방식

- 장점
  - 동일한 Process의 자원을 Thread들은 서로 공유(stack 영역을 제외한 모든 memory)
    - 자원 할당을 위한 System call이 적음
    - 통신의 부담이 적음(Context Switching이 빠름)
    - 통신 비용이 줄음(IPC보다 통신 부담이 적음)
- 단점
  - Thread간의 자원 공유로 인한 문제 발생
    - Synchronization Issue(동기화 문제)
    - 잘못된 변수 공유로 인한 오류 발생
    - 하나의 Thread로 인한 전체 Process 영향 

<br />

---


##### 3. Process vs Thread

- memory의 공유 여부
- 영향성(동기화 문제) vs 자원의 효율성, 처리량

<br />

---
