---
title:  "Vuex(Store)"
excerpt: "[Vue] About Vuex(Store)..."

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2023-07-31
---


## 0. 들어가면서

- 프로젝트 진행 중에 props로 컴포넌트 간에 데이터를 전달하기보다는 Vuex의 Store를 활용하여 내부의 데이터들을 관리하면서 다시 한 번 정리하고자함
- Vuex의 특징과 사용성의 이점에 대하여 알아보고자함

---

#### Vuex(Store) ??

> Vuex는 Vue.js 애플리케이션에 대한 상태 관리 패턴 + 라이브러리 입니다. 애플리케이션의 모든 컴포넌트에 대한 중앙 집중식 저장소 역할을 하며 예측 가능한 방식으로 상태를 변경할 수 있습니다. 또한 Vue의 공식 devtools 확장 프로그램 (새 창을 엽니다)과 통합되어 설정 시간이 필요 없는 디버깅 및 상태 스냅 샷 내보내기/가져오기와 같은 고급 기능을 제공합니다


![image info](/assets/img/vuex_img.png)
<img src="/assets/img/vuex_img.png" alt="" width="0" height="0">

- Vue 개발에서 모든 컴포넌트들에 대한 중앙 집중식 저장소의 역할, 관리를 담당
- Vuex가 없다면 Component간의 data를 부모, 자식간에 props, emit을 사용하여 전달해야함
- Vuex를 활용하여 데이터를 따로 전달하는 것이 아닌 모든 Component에서 직접 접근이 가능하도록 함
- 상태유지, 관리에 매우 좋음

<br />

#### Store의 구성

- State
- Getters
- Mutations
- Actions

<br />


---

### 1. State

- 데이터의 상태를 관리하는 부분
- Component 간에 공유하며 사용, 관리하는 data
- 전역적으로 data를 사용 가능
  
```javascript
//Store
const state = {
    walletConnected : ((sessionStorage.getItem('chainType') != null) ? true : false)
    
}
```

- 초기에 sessionStorage를 확인하여 state를 true, false로 상태가 정해짐

<br />

---

### 2. Mutations

- State의 상태 변경을 관리하는 부분
- 동기 처리
- commit('함수명', param)를 통해서 호출
  

```javascript
//Store
const mutations = {
    connectedChange(state, data){
        state.walletConnected = data
    }
}

context.commit('connectedChange', data);
```

- connectedChange() 호출 시 param 값에 따라 state의 data를 변경함

<br />

---

### 3. Actions

- Mutations를 실행시키는 부분
- 비동기 처리
- dispatch('함수명', param)를 통해서 호출

```javascript
//Store
const actions = {
    async connectedSync(context) {
        try {
            const data = (sessionStorage.getItem('chainType') != null) ? true : false
            context.commit('connectedChange', data);
        } catch (e) {
            console.log(e)
        }
    }
}

//Component
this.$Store.dispatch("connectedSync");
```

- conenctedSync안에서 connectedChange를 호출하기 때문에 connectedSync가 diispatch로 호출 되면 sessionStorage의 상태에 따라 state의 data를 변경함


<br />

---

### 4. Getters

- Getters에 정의하여 Component에서 state의 data를 불러옴
- 모든 Component에서 Computed의 getters를 사용하면 state의 data를 동기화하여 계속 가져올 수 있음
- this.$Store.getters['경로명/함수명'];

```javascript
//Store
const getters = {
    getWalletConnected(state) {
        return state.walletConnected;
    }
}

//Component
computed: {
    ...mapGetters({
      connected: "getWalletConnected",
    }),
}
```

- computed를 통하여 변경되는 Store의 state의 data를 가져올 수 있음


<br />

---

### Full Example Code

```javascript
//Store
const state = {
    walletConnected : ((sessionStorage.getItem('chainType') != null) ? true : false)
    
}

const getters = {
    getWalletConnected(state) {
        return state.walletConnected;
    }
}

const mutations = {
    connectedChange(state, data){
        state.walletConnected = data
    }
}

const actions = {
    async connectedSync(context) {
        try {
            const data = (sessionStorage.getItem('chainType') != null) ? true : false
            context.commit('connectedChange', data);
        } catch (e) {
            console.log(e)
        }
    }
}

export default {
    state,
    getters,
    mutations,
    actions
}



//App.vue
window.addEventListener("load", function () {
    this.$Store.dispatch("connectedSync");
}.bind(this),1000);
```

- 해당 Store를 통해서 SessionStorage값의 변경에 따라 state의 walletConnected를 변경함
- 페이지 새로 로딩될 때마다 App.vue에 dispatch를 호출하여 sync를 맞춤
- computed를 통해서 해당 Component에서 wallet이 연결되어있는지 페이지를 로딩될 때마다 확인 가능

---

#### Reference

- https://v3.vuex.vuejs.org/kr/