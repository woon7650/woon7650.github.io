---
title:  "ComputedCaching"
excerpt: "[Vue] ComputedCaching"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2023-07-27
---


#### 0. 들어가면서

- 현재 프로젝트의 진행하면서 실시간 가격폭 변동, 블록체인 상 정보 감지 및 여러 데이터의 실시간 변화 감지 관련 부분을 처리하면서 Vue의 Computed와 Watch에 대해 다시 한번 정리하고자함
- computed, watch, methods의 data 변경에 따른 차이를 알아보면서 React에 연관 지어보고자함

---

#### Caching ??

- 오랜시간이 걸리는 작업의 결과를 저장해서 시간과 비용을 필요로 회피하는 기법
- 캐시 영역으로 데이터를 가져와서 접근하는 방식

---

#### Example Code

```javascript
<template>
    <div>
        <span>{{ userName }}</span>
        <span>{{ viewUserName}}</span>
        <span>{{ returnUserName() }}</span>
    </div>
</template>

<script>
import { mapGetters } from "vuex";

export default {
    name: "Example",
    data(){
        return{
            userName : null;
        }
    }
    //watch
    watch: {
        user(newVal, oldVal) {
            if (newVal) {
                this.userName = newVal.userName
            }
        },   
    },
    //computed
    computed: {
        ...mapGetters({
            user: "user/getUser"
        }),
        userName(){
            return this.user.userName;
        }
    },
    //methods
    methods: {
        returnUserName(){
            return this.user.userName;
        }
    }
}
</script>
```

<br />

---

### 1. Computed

- Store에서 불러오는 user의 값이 바뀔 때마다 userName의 값이 변함
- <mark style="background-color:#cccccc">실시간</mark>으로 바뀐 user의 userName을 return
- return이 필수로 사용됨
- Caching O
  

<br />

---

### 2. Watch

- Store에서 불러오는 user의 값이 바뀔 때마다 바뀐 userName 변수를 new value로 변경
- <mark style="background-color:#cccccc">실시간</mark>으로 바뀐 user를 감지하여 userName을 변경
- 조건문을 사용해서 원하는 newValue에 따라 사용자가 원하는 로직 생성이 가능함
- Caching O

<br />

---

### 3. Methods

- returnUserName() method를 호출하는 시점의 Store에 저장되어있는 user의 userName을 return
- 해당 코드에서는 returnUserName()은 해당 Component를 <mark style="background-color:#cccccc">Rerendering</mark>시에 실행됨
- Event와 함께 사용하는 적합(event 발생 시점의 data return)
- Caching X

<br />

---
#### Reference
- https://vuejs.org/guide/essentials/computed.html
- https://velog.io/@leehaeun0/Vue-method-computed-watch