---
title:  "Async/Await & Promise"
excerpt: "[Javascript] About Async/Await & Promise ... "

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2023-08-21

---

### 0. 들어가면서

- 비동기명령의 두가지 방법인 <mark style="background-color:#cccccc">Promise</mark>와 <mark style="background-color:#cccccc">Async/Await</mark>의 차이점에 대해 자세히 알아보고자 합니다

---

### 1. Schedule Task in Javascript

> Javascript는 <mark style="background-color:#cccccc">MicroTask Queue</mark>, <mark style="background-color:#cccccc">Callback Queue</mark> 두 가지 방법으로 비동기 작업을 예약합니다.


![image info](/assets/img/async_promise.gif)
<img src="/assets/img/async_promise.gif" alt="" width="0" height="0">

<br />

##### 1.1 MicroTask Queue

> MicroTask Queue는 현재 작업 이후에 실행되는 작업의 대기열입니다.

- MicroTask Queue은 Event Loop에서 처리됩니다.
- MicroTask Queue은 현재 작업이 완료된 후 처리됩니다.
- MicroTask Queue은 Callback Queue의 다음 작업으로 이동하기 전에 Javascript 엔진에 의해 처리됩니다.

<br />

##### 1.2 Callback Queue

> Callback Queue는 현재 작업 이후에 실행되는 작업 대기열입니다.

- Callback Queue는 해당 MicroTask Queue과 동일한 Event Loop에서 처리됩니다.
- Callback Queue은 MicroTask Queue가 비워진 후 처리됩니다.
- Callback Queue은 MicroTask Queue의 모든 작업을 실행한 후에 Javascript 엔진에 의해 처리됩니다.


<br />

##### 1.3 MicroTask Queue & Callback Queue

```javascript
<script>
export default {
  mounted() {

    console.log('start');

    setTimeout(function(){
        console.log('setTimeout')
    },0);

    Promise.resolve().then(function(){
        console.log('promise resolve');
    });

    console.log('end');
  }
  methods: {

  }
};

</script>
```

<br />

Console Log
```javascript
start
end
promise resolve
setTimeout
```

- setTimeout 은 Callback Queue에 추가됨
- Javascript 엔진은 현재 모든 작업의 모든 코드 실행을 마친 후 setTimeout 실행
- Promise.resolve 은 MicroTask Queue에 추가됨
- Javascript 엔진은 Callback Queue로 이동하기 전에 MicroTask Queue의 모든 작업을 실행함

<br />

---

### 2. Async / Await & Promise

> Javascript는 <mark style="background-color:#cccccc">Async/Await</mark>, <mark style="background-color:#cccccc">Promise</mark> 두 가지 방법으로 비동기 작업을 처리합니다.

<br />

##### 2.1 Promise

> 비동기 동작을 완료 or 실패로 이끌어내는 객체

- pending, fulfilled, rejected
- 완료 상태
  - fulfilled + data
  - rejected + error

- Promise가 생성되고 비동기 작업이 시작하면 Promise 생성 이후의 코드는 동기적으로 계속 진행됨
- Promise가 fulfill, reject되면 연결된 Callback 함수가 MicroTask Queue에 추가됨
- Promise 생성 이후의 모든 코드는 Promise에 연결된 콜백 함수가 실행되기 전에 실행됨

<br />

##### 2.2 Async / Await

> Promise의 syntax sugar

- Promise가 해결되거나 거부될 때까지 비동기 함수의 실행을 일시 중지(Await)
- Promise가 해결되면 비동기 함수의 실행이 재개되고 결과가 반환됨(data or error)
- 같은 async내에서 await간의 순서 보장
  - Async/Await 사용 해당 함수는 정지된 상태로 MicroTask Queue에 추가되며 CallBack Queue가 비면 실행됨

<br />



##### 2.3 Promise & Async / Await

> Difference : ExecutionContext(실행 컨텍스트)

- ExecutionContext를 CallStack에 쌓아올린 후 실행하여 환경과 순서를 보장합니다.
- 실행 순서를 보장하는가 안하는가

<br />


---

##### Reference

- https://medium.com/version-1/difference-between-promise-and-async-await-95e453182f55
- https://www.tutorialspoint.com/what-is-difference-between-microtask-queue-and-callback-queue-in-asynchronous-javascript