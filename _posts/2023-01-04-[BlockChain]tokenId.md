---
title:  "[BlockChain]Opensea LazyMinting Token Id에 대하여"
excerpt: "About Opensea LazyMinting Token Id..."

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2023-01-04
---

## TokenId 

---

### Minting 과 Lazy minting의 차이

> Minting : smart contract을 통해 on-chain상에서 nft 생성 시 지정되는 nft의 고유 id <br/>
> Lazy Minting : off-chain상에서 metadata 생성 후 voucher 생성 시 지정되는 nft의 고유 id
<br />

---

### Lazy minting token id 바이트 할당
- nft id와 token id는 다른 개념임
- token id는 32 Byte로 이루어져 있음
- nft id는 7 Byte로 이루어져 있음
- 창작자(아티스트)의 wallet address, nft id, 해당 nft의 공급된 개수를 포함하고 있음
- nft의 공급된 개수
    - ERC-721 : 1개 
    - ERC-1155 : minting한 nft 개수

> **앞자리 20 byte** : 창작자(아티스트)의 wallet address<br/>
> **중간 7 byte** : nft의 고유 id(index)<br/>
> **마지막 5 byte** : 해당 nft의 공급된 개수<br/>


---
### Opensea token id

- opensea의 일반적인 minting은 ERC-1155의 LazyMinting으로 진행됨
- 구매자가 nft를 구매하는 시점 전까지 off-chain 사용(db 이용)
- 구매자가 nft를 구매하는 시점 이후로 on-chain 사용(smart contract 이용)

---
<br/>

**Example) Opensea Lazy Minting Description .PNG**

<br/>

![image info](/assets/img/opensea_tokenId.png)
<img src="/assets/imgopensea_tokenId.png" alt="" width="0" height="0">


TokenId(10진수) : 71577999342193181516787717611109606829694518788112888032663196598076478324737
TokenId(16진수) : 9E3FB64223BEE3FB2DEFDEC5C1190768F448EB4B000000000000010000000001

<br/>

- **앞자리 20 byte** : 0x9E3FB64223BEE3FB2DEFDEC5C1190768F448EB4B<br/>
- **중간 7 byte** : 00000000000001<br/>
- **마지막 5 byte** : 0000000001<br/>

<br/>

- 창작자(아티스트) Address : 0x9E3FB64223BEE3FB2DEFDEC5C1190768F448EB4B<br/>
- nft id : 1<br/>
- nft totalSupply : 1<br/>

---
### Token id 생성 구현

```javascript
async creatorNftId(address){
  var nftAmount = await this.selectNftCountWithAddress(address);
  //address로 backend에 요청을 보내서 db에서 해당 address의 nft 개수 확인
  var creatorNftAmount = (nftAmount + 1).toString();
  return creatorNftAmount.padStart(14,'0');
},
creatorNftQuantity(amount){
  var creatorNftQuantity = (amount).toString();
  return creatorNftQuantity.padStart(10,'0');;
},
async generateVoucherTokenId(amount){
  //20 Byte wallet address
  var creatorAddress  = (await this.getEthWallet())[0].toString();

  var creatorNftId = await this.creatorNftId(creatorAddress);

  var creatorNftQuantity = this.creatorNftQuantity(amount);

  var tokenId = creatorAddress + creatorNftId + creatorNftQuantity;
  return (BigInt(tokenId)).toString(10);
}


var voucherTokenId = await this.generateVoucherTokenId(amount);
```

---
