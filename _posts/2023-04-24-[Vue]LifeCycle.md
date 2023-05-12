---
title:  "[Vue] LifeCycle(생명 주기)"
excerpt: "LifeCycle(생명 주기)"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2023-04-24
---


## LifeCycle(생명 주기)

<br/>


- Vue의 instance나 component가 생성될 때부터 소멸 될까지를 단계를 뜻함<br />
- 라이프사이클의 각 단계에서 실행되는 methods은 라이프사이클 훅(LifeCycle Hooks)라고 함

<br />

---
## LifeCycle Hooks

<br/>

- Vue instance나 component들이 생성, 소멸되기까지의 단계를 말하며 각 단계에서 실행되는 methods을 라이프사이클 훅이라고 함

<br/>

---

### Vue LifeCycle Flowchart

<br/>

![image info](/assets/img/lifecycle.png)
<img src="/assets/img/lifecycle.png" alt="" width="0" height="0">

<br/>

---

### LifeCycle Hooks Methods

<br/>

- Creation
- Mounting
- Updating
- Destruction

---

### Creation( Initialize Component ) : 서버렌더링에서도 지원되는 훅(클라이언트, 서버단 렌더링 모두)

<br/>

>beforeCreate
 - DOM에 접근할 수 없음(Component가 DOM에 추가되기 전에 실행되는 라이프사이클 훅)
 - data, events가 세팅되지 않은 시점에서 접근 시도시 error 발생

>created
 - data, events가 활성화된 이후 시점으로 접근 가능
 - mount lifecycle hook 전 단계이기 때문에 DOM에는 접근 불가


---

### Mounting( Insert DOM ) : 서버렌더링이 실행되는 동안은 실행되지 않음(초기 렌더링 전후에 component에 엑세스 할 수 있다)

<br/>

>beforeMount
 - 초기 렌더링이 일어나기 직전에 실행

>mounted
 - Component, template, DOM에 접근할 수 있으며 DOM에 접근해서 수정이 가능하다
 - 자식 Component -> 부모 Component 순서로 hook이 실행된다


---

### Updating( Re-rendering ) : Component에서 사용하는 반응형 속성들이 update 혹은 re-render 때마다 호출(component 속성들의 변경 시점을 알아야할 때는 watch나 computed 사용)

<br/>

> beforeUpdate
 - component 데이터가 변경되어 DOM이 패치되고 re-render 되기 직전에 실행

> updated
 - component 데이터가 변경되어 DOM이 re-render 된 후 실행
 - property가 변경된 후 DOM에 접근 시 사용

---

### Destruction( Component Collapse ) : Component가 해체, 분리되고 DOM에서 제거될 때 실행

<br/>

>beforeDestroy
 - component 해체, 분리 직전에 실행되고 component는 여전히 있음
 - component에 걸려있는 event listener를 정리하는데 사용

>destroyed
 - component가 해체, 분리된 후 실행



---


```javascript
<script>
export default {
  data() {
    return {
    };
  },
  beforeCreate() { 
    
  },
  created() {

  },
  beforeMount() {

  },
  mounted() {

  },
  beforeUpdate() {

  },
  update() {

  },
  beforeDestroy() {

  },
  destroyed() {

  },

  methods: {

  }
};
</script>
```

---

### Vue LifeCycle Hook 에서의 비동기 처리 시 주의할 것

<br/>

권장하지 않는 방법
> created Hook & mounted Hook
> - async를 붙여도 비동기로 바뀌지 않음
> - created, mounted hook에 async가 선언되어 있어도  await 함수가 다 이행될 때까지 created에서 기다려 주지 않음
> - 시간이 오래 걸리는 비동기 처리 함수들은 다 처리가 되기 전에 다음 훅으로 넘어감

<br/>

권장하는 방법
> created Hook & mounted Hook
> - async method로 따로 만들어서 사용
> - 가급적이면 오래걸리는 비동기 처리는 DOM 랜더링 완료 후 속성, 값 반영이 안 될 수 있음

```javascript
import { fetchTableData, fetchTimeData } from '@/api/table'
<script>
export default {
  data() {
    return {
    };
  },
  created() {
    this.fetchTimeData().then((res) => {
      console.log(res)
    })
  },
  mounted() {
    this.fetchData().then((res) => {
      console.log(res)
    })
  },

  methods: {
    async fetchData(){
        var resultData = await fetchTableData();
        return resultData;
    },
    async fetchTime(){
        var resultData = await fetchTimeData();
        return resultData;
    }
  }
};
</script>
```

