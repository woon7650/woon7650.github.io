---
title:  "custom env"
excerpt: "[Vue] custom env"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2023-12-20

---


#### 0 .env

> Vue 프로젝트 내에서 전역 변수의 설정 및 사용 시 필요한 환경 변수 파일 

- 선언 형식 : key = value
- 사용 형식 : process.env.변수명
  
<br />

---

### 1 .env 파일의 종류 및 생성

.env.development
```javascript
NODE_ENV = 'development'

VUE_APP_BASE_API= your_base_api

VUE_APP_BASE_URL= you_base_url


VUE_APP_ETHEREUM_CONTRACT_ADDRESS = your_mainnet_contract_address
VUE_APP_ETHEREUM_CHAINID = '1'
```

.env.production
```javascript
NODE_ENV = 'production'

VUE_APP_BASE_API= your_base_api

VUE_APP_BASE_URL= you_base_url


VUE_APP_ETHEREUM_CONTRACT_ADDRESS = your_mainnet_contract_address_v1
VUE_APP_ETHEREUM_CHAINID = '1'
```

.env.testnet
```javascript
NODE_ENV = 'testnet'

VUE_APP_BASE_API= your_base_api

VUE_APP_BASE_URL= you_base_url


VUE_APP_ETHEREUM_CONTRACT_ADDRESS = your_testnet_contract_address
VUE_APP_ETHEREUM_CHAINID = '11155111'
```

- 기존에는 mainnet 네트워크의 개발, 운영 환경 두 개를 사용하였지만 신규로 testnet 네트워크의 개발 환경을 추가로 생성해야 되는 상황이 생겼습니다.
- 현재 NFT 프로젝트를 진행하면서 testnet 환경, mainnet 서버(개발 환경, 운영 환경 -> 같은 DB 사용)를 사용하면서 3개의 환경 변수 파일을 필요로 하게 되었습니다.
- mainnet 네트워크를 사용하며 실제 운영되는 운영 환경, mainnet 네트워크를 사용하며 개발용으로 사용되는 개발 환경, testnet 네트워크를 사용하며 개발용으로 사용되는 개발 환경로 존재합니다.
- 따라서 기존에 development, production으로만 나눠서 사용하던 .env 파일에 custom된 .env 파일이 추가되었습니다.

<br />

---


### 2 .env 파일의 사용

package.json
```javascript
{
  "scripts": {
    "serve": "vue-cli-service serve",
    "serve:testnet": "vue-cli-service serve --mode testnet",
    "build:prod": "vue-cli-service build",
    "build:testnet": "vue-cli-service build --mode testnet"
  }
}
```

- npm run serve : <mark style="background-color:#cccccc">.env.development</mark> 환경을 이용해서 development mode로 로컬 서버 가동
- npm run serve:testnet : <mark style="background-color:#cccccc">.env.testnet</mark> 환경을 이용해서 testnet mode로 로컬 서버 가동
- npm build:prod : <mark style="background-color:#cccccc">.env.production</mark> 환경을 이용해서 production mode로 build
- npm build:testnet : <mark style="background-color:#cccccc">.env.testnet</mark> 환경을 이용해서 testnet mode로 build

<br />


---

### Refernce 
- https://truehong.tistory.com/135
- https://taemy-sw.tistory.com/12
- https://wouldyou.tistory.com/82
- https://joshua1988.github.io/vue-camp/deploy/env-setup.html#vue-cli-3-x-%E1%84%87%E1%85%A5%E1%84%8C%E1%85%A5%E1%86%AB%E1%84%8B%E1%85%B4-%E1%84%92%E1%85%AA%E1%86%AB%E1%84%80%E1%85%A7%E1%86%BC-%E1%84%87%E1%85%A7%E1%86%AB%E1%84%89%E1%85%AE-%E1%84%91%E1%85%A1%E1%84%8B%E1%85%B5%E1%86%AF-%E1%84%8C%E1%85%A5%E1%86%B8%E1%84%80%E1%85%B3%E1%86%AB