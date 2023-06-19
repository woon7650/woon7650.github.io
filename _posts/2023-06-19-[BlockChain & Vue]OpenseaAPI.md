---
title:  "[BlockChain & Vue] Opensea NFT API"
excerpt: "About Opensea NFT API"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2023-06-19
---


### 0. 들어가면서

<br />

##### Opensea NFT API를 사용한 목적

- mainnet network에서 사용자가 소유한 NFT의 목록을 조회
- mainnet network에서 해당 NFT의 정보를 조회
- 내부 플랫폼에서 민팅한 NFT를 외부 플랫폼인 Oensea에서 거래
- Opensea에서 민팅한 NFT를 내부 플랫폼에서 거래
- 거래 외에 NFT 관련 이벤트를 발생시켜서 NFT의 정보를 변경
- alchemy, infura 등 NFT API는 많이 지원하지만 거래에 필요한 API는 Opensea에서만 제공

---

### 1. Opensea NFT API

<br />

> OpenSea NFT API는 OpenSea에 등록된 NFT에 대한 데이터에 액세스하는 데 사용할 수 있는 Tool입니다. OpenSea API는 RESTful API이며, JSON을 통해 데이터를 제공합니다. OpenSea API를 사용하면 NFT의 메타데이터, 소유자, 판매 기록과 같은 NFT에 대한 정보를 가져올 수 있고 NFT 목록을 만들고 관리하고, NFT를 보내고 받고, NFT 가격을 추적할 수도 있습니다.

---

### 2. 환경 세팅

<br />

##### 2.1 Version

- Opensea API v1
- Opensea API v2(Opensea.js v 6.0.5)
- npm v 6.14.13
- @vue/cli 5.0.4
- chrome metamask
- Opensea API KEY

<br />

##### 2.2 Installation

- npm install Opensea-js
- npm install ethers

<br />

##### 2.3 Reference

- <mark style="background-color:#cccccc">Opensea API KEY는 mainnet만 제공하고 testnet용은 제공하지 않음</mark>

---

### 3. Opensea API v1

<br />

```javascript
const axios = require('axios');

//Opensea mainnet
const APIUrl = 'https://API.Opensea.io/API/v1'
const APIKey = YOUR_OPENSEA_API_KEY


export default {
  methods: {
    //fetching NFT list owned by ownerAddress in mainnet
    async getOwnerNFTs(ownerAddress) {
      try {
        const response = await axios.get(`${APIUrl}/assets`, {
          params: {
            owner: ownerAddress,
            order_direction: 'desc',
            offset : 0,
            limit: 10,
            include_orders: false
          },
          headers: {accept: 'application/json', 'X-API-KEY': APIKey}
        });
        console.log('소유자의 NFT 목록:', response.data.assets);
      } catch (error) {
        console.log(error)
        console.error('NFT를 가져오는 중 오류가 발생했습니다:', error.message);
      }
    },

    //fetching NFT detail from mainnet
    async getNFTInfo(contractAddress, tokenId) {
      try {
        const response = await axios.get(`${APIUrl}/asset/${contractAddress}/${tokenId}`);
        const NFTInfo = response.data;
        console.log('NFT 정보:', NFTInfo);
        return NFTInfo;
      } catch (error) {
        console.error('NFT 정보를 가져오는 중 오류가 발생했습니다:', error.message);
      }
    }

  }
}
```
<br />

##### 3.1 Opensea API v1을 사용한 이유
- Opensea sdk v2는 collection 단위로 NFT를 검색해옴
- 내부 플랫폼은 collection 단위가 아닌 wallet address 기반의 각각의 NFT로 이루어져있음
- v1을 사용하여 collection 단위가 아닌 각각의 NFT를 개별적으로 조회함

---
### 4. Opensea API v2

```javascript
import { OpenSeaPort } from 'Opensea-js';
import { ethers } from 'ethers';


//const provider = new ethers.providers.JsonRpcProvider(YOUR_INFURA_PROJECT_ID);
const provider = new ethers.providers.Web3Provider(window.ethereum);

const axios = require('axios');

const APIUrl = 'https://API.Opensea.io/API/v2'
const APIKey = YOUR_OPENSEA_API_KEY


export default {
  data: function () {
    return {
        seaport : null
    };
  },
  methods: {
    //creating seaport instance
    async createSeaportInstance() {
      try{
        this.seaport = new OpenSeaPort({
          networkName: "mainnet",
          provider : provider,
          APIKey: APIKey,
        });
        return this.seaport
      }catch(error){
        console.log(error)
      }
        console.log(this.seaport)
    },
    async fetchAsset(contractAddress, tokenId) {
      await this.createSeaportInstance();
      const asset = await this.seaport.API.getAsset({ tokenAddress: contractAddress, tokenId: tokenId });
      console.log('assest info : ')
      console.log(asset);
    },
    //sell external NFT in our platform
    async sellExternalNFT(tokenId, tokenAddress, accountAddress, price) {
      try {
        const listing = await this.seaport.createSellOrder({
          asset: {
            tokenId: tokenId,
            tokenAddress: tokenAddress,
          },
          accountAddress: accountAddress,
          startAmount: price,
          endAmount: price,
          expirationTime: 0
        });
        console.log(listing)
        console.log('NFT 판매가 성공적으로 등록되었습니다.');
      } catch (error) {
        console.error('NFT 판매 등록 중 오류가 발생했습니다:', error);
      }

    },
  }
}
```
<br />

##### 4.1 Opensea API v2를 사용하는 순서

- JsonRpcProvider(infura) 또는 Web3Provider(wallet)을 이용해서 provider를 생성
- OpenseaPort 클래스의 instance를 생성
- OpenseaPort 인스턴스를 이용해서 Opensea NFT API와 상호작용
- 제공해주는 NFT 관련 API들 이용

<br />

##### 4.2 Opensea API v2를 사용하는 이유

- v1에서는 지원하지 않는 NFT 관련 거래 기능을 이용
- 지갑 주소를 기반으로 v1을 사용해서 해당 NFT의 정보를 가져오고 collection을 알아낸 후 collection 기준으로 해당 NFT에 이벤트를 발생시킬 수 있음

<br />

##### 4.3 Opensea API v2가 제공하는 기능

- Fetching Assets
- Checking Balances and Ownerships
- Making Offers
- Bidding on ENS Short Name Auctions
- Offer Limits
- Making Listings / Selling Items
- Fetching Orders
- Buying Items
- Accepting Offers
- Transferring Items or Coins (Gifting)

---
### 5. Opensea SDK, Opensea JS

|구분|Opensea JS|Opensea SDK|
|:--:|:--:|:--:|:--:|
|언어|Javascript|Python|
|유연성|낮음|높음|
|사용 편의성|높음|낮음|
|제어|낮음|높음|

---
#### Reference
- https://github.com/ProjectOpenSea/opensea-js