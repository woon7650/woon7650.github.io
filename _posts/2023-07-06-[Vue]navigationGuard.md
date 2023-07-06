---
title:  "Navigation Guard"
excerpt: "[Vue] About Navigation Guard"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2023-07-06
---


## 0. 들어가면서


##### Navigation Guard

> *Navigation Guards* provided by Vue router are primarily used to guard navigations either by redirecting it or canceling it. There are a number of ways to hook into the route navigation process: globally, per-route, or in-component.

##### Guard Types

- Global Guard
- Route Guard
- Component Guard

##### Parameter
- to : 이동할 Route 객체
- from : 현재 Route 객체
- next : Guard의 결과로 어떤 action을 할지 결정하는 method(Hook 종료를 위해 필수적으로 사용)

<br />

---


# 1. Global Guard

- <mark style="background-color:#cccccc">어플리케이션 전역에서 동작</mark>


## 1.1 Global Guard Types

- ##### 1.1.1 BeforeEach
  - 모든 Route 전환 이전에 실행되는 전역 Guard
  - 사용자가 custom해서 다음 action을 정할 수 있음
  - 주로 전역적으로 이동 전에 검사를 해야하는 부분들을 처리하기 위해 사용(로그인, 인증...)
  - parameter : to, from, next

<br />

- ##### 1.1.2 AfterEach
  - 모든 Route 전환 이후 실행되는 전역 Guard
  - beforeEach에서 이미 로직을 처리 후에 실행됨
  - parameter : to, from

<br />

- ##### 1.1.3 BeforeResolve

  - 비동기 데이터 로딩이 완료된 후 실행되는 전역 Guard
  - 로직 중에 비동기 데이터를 이용해야 할 때  사용
  - parameter : to, from, next

<br />


- ##### 1.1.4 AfterResolve

  - 비동기 데이터 로딩이 완료되고 초기화가 완료된 후 실행되는 전역 Guard
  - parameter : to, from

<br />

## 1.2 Global Guard Example Code

```javascript
import router from '@/router'
import store from '@/store'
import eth from '@/mixin/Ethereum.vue'
import klay from '@/mixin/Klaytn.vue'


router.beforeEach((to, from, next) => {
    const userName = store.getters.name
    const isAdmin = store.getters.admin
    if(userName && isAdmin){
        next()
    }else if(userName && !isAdmin){
        next('/unauthorize')
    }else{
        next('/login')
    }
})
router.beforeResolve(async (to, from, next) => {
    var address = null
    if (sessionStorage.getItem('chainType') == 'eth') {
      address = (await eth.methods.getEthWallet())[0];
      next()
    } else if (sessionStorage.getItem('chainType') == 'klay') {
      address = (await klay.methods.getKlayWallet())[0]
      next()
    }
  
})
router.afterEach((to, from) => {
    console.log(to.path + "로 이동")
})
router.afterResolve(async (to, from) => {
    console.log("데이터 초기화 완료")
})
```

<br />

---


# 2. Router Guard

- <mark style="background-color:#cccccc">Route 설정 객체에 직접 정의</mark>

## 2.1 Router Guard Types

- ##### 2.1.1 beforeEnter

  - 특정 Route 진입 직전에 실행되는 Guard

<br />


## 2.2 Router Guard Example Code

```javascript
export const router = new VueRouter({
	routes : [
    	{
        	path: '/home',
          component: Home,
          beforeEnter: function(to, from, next) {
          	// 인증을 위한 로직 작성
          }
      },
    ]
})
```

<br />

---


# 3. Component Guard

- <mark style="background-color:#cccccc">Route Component 내부에 정의</mark>

## 3.1 Component Guard Types

- ##### 3.1 beforeRouteEnter

  - Component가 라우트에 진입하기 전에 실행되는 가드
  - 'this'에 접근할 수 없음

<br />

- ##### 3.2 beforeRouteUpdate

  - Component가 현재 라우트에서 업데이트되기 전에 실행되는 가드
  - 'this'에 접근할 수 있음

<br />

- ##### 3.3 beforeRouteLeave

  - Component가 현재 라우트에서 떠나기 전에 실행되는 가드
  - 'this'에 접근할 수 있음

<br />



## 3.2 Component Guard Example Code

```javascript
const userSetting = {
	beforeRouterEnter(to, from, next) {
    console.log('Detected : component ready to entered Route')
	},
	beforeRouterUpdate(to, from, next) {
    console.log('Detected : component ready to upadte detected')
	},
	beforeRouterLeave(to, from, next) {
    console.log('Detected : component ready to leave Route detected')
	}
}
```

<br />

---


# 4. Navigation Guard Flow

<br />

1. Navigation triggered 
2. Call beforeRouteLeave(Component Guard) -> deactivated components
3. Call beforeEach(Global Guard)
4. Call beforeRouteUpdate(Component Guard) -> reused components
5. Call beforeEnter(Route Guard)
6. Resolve async Route Components
7. Call beforeRouteEnter(Component Guard)
8. Call befoerResolve(Global Guard)
9. Call navigation confirmed
10. Call afterEach(Global Guard)
11. DOM updates triggered
12. Call beforeRouteLeave(Component Guard)

<br />

---

## Reference

- https://github.com/ProjectOpenSea/opensea-js
https://router.vuejs.org/guide/advanced/navigation-guards.html#in-component-guards

---