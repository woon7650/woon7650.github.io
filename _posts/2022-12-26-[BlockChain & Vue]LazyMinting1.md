---
title:  "[BlockChain & Vue]Lazy Minting에 대하여(Part1)"
excerpt: "About Lazy Minting(Part1)..."

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2022-12-26
---

## Lazy Minting(Part1)
<br />

### 정의
- 창작자(아티스트)가 만든 NFT를 누군가 NFT를 구매할 때까지 블록체인 네트워크 상에 등록하지 않습니다.
- Lazy Minting으로 발행된 NFT는 구매자가 NFT를 구매하는 시점에 창작자(아티스트)가 NFT를 발행하는 데 발생하는 가스 비용과 NFT 비용을 동시에 지불합니다.
- NFT 발행에 드는 가스 비용을 구매자가 지불하면서 NFT는 블록체인 네트워크 상에 등록됩니다.

---

### 과정(순서)
- 창작자(아티스트)는 smart contract를 이용하여 NFT를 발행하고 동시에 판매등록을 합니다.
- 창작자(아티스트)는 Lazy Minting의 프로세스 승인을 위해 지갑, NFT 세부 정보를 자세히 설명하는 개인 서명을 제공합니다.
- 구매자는 NFT를 구매하고 창작자(아티스트)가 NFT 발행에 필요한 가스 비용가 NFT의 자체 비용을 동시에 지불하고 NFT를 오프체인에서 온체인으로 등록합니다.

---

### 공유(플랫폼 간)
- 대표적인 Lazy Minting 플랫폼인 Rarible, Opensea에서 Lazy Minting으로 각각 NFT를 발행했을때 서로의 Platform에서는 판매 및 List 할 수 없습니다.
- Lazy Minting은 서명 및 NFT의 정보를 구매자가 구매하기 전까지는 온체인이 아닌 오프체인(DB)상에 존재하기 때문에 다른 Platform에서는 판매 및 List 할 수 없습니다. 

---

### Smart Contract
```solidity
struct NFTVoucher {
  uint256 tokenId;
  uint256 minPrice;
  string uri;
  bytes signature;
}
```
- tokenId : 온체인에 발행될 tokenId
- uri : 해당 nft의 metadata가 담겨있는 ipfs uri
- minPrice : 창작자(아티스트)로 부터 nft를 구매하기 위한 최소 금액
- signature : 창작자(아티스트)의 지갑으로 만들어진서명

```solidity

  function redeem(address redeemer, NFTVoucher calldata voucher) public payable returns (uint256) {
    address signer = _verify(voucher);
    require(hasRole(MINTER_ROLE, signer), "Signature invalid or unauthorized");

    require(msg.value >= voucher.minPrice, "Insufficient funds to redeem");

    _mint(signer, voucher.tokenId);
    _setTokenURI(voucher.tokenId, voucher.uri);

    _transfer(signer, redeemer, voucher.tokenId);

    address payable tokenOwner = payable(signer);
    tokenOwner.transfer(msg.value);       

    return voucher.tokenId;
  }

  function _hash(NFTVoucher calldata voucher) internal view returns (bytes32) {
    return _hashTypedDataV4(keccak256(abi.encode(
      keccak256("NFTVoucher(uint256 tokenId,uint256 minPrice,string uri)"),
      voucher.tokenId,
      voucher.minPrice,
      keccak256(bytes(voucher.uri))
    )));
  }

  function _verify(NFTVoucher calldata voucher, bytes memory signature) 
  internal view returns (address) {
    bytes32 digest = _hash(voucher);
    return ECDSA.recover(digest, signature);
  }
```

---

### JavaScript
```javascript
async createVoucher(tokenId, uri, minPrice) {
  const provider = new ethers.providers.Web3Provider(window.ethereum)
  const signer = provider.getSigner()
  console.log(signer)
  const voucher = { tokenId, uri, minPrice }
  const domain = await this._signingDomain()
  const types = {
    NFTVoucher: [
      {name: "tokenId", type: "uint256"},
      {name: "minPrice", type: "uint256"},
      {name: "uri", type: "string"},  
    ]
  }
  const signature = await signer._signTypedData(domain, types, voucher)
  return {
    ...voucher,
    signature,
  }
},
async _signingDomain() {
  if (this.domain != null) {
  	return this.domain
  }
  const chainId = await web3.eth.getChainId()
  console.log(typeof(chainId))
  this.domain = {
  	name: SIGNING_DOMAIN_NAME,
  	version: SIGNING_DOMAIN_VERSION,
  	chainId: BigNumber.from(chainId),
  	verifyingContract: this.eip712ContractAddress
  }
  return this.domain
}
```

- _signTypedData(domain, types, voucher) : ether.js에서 제공하는 함수
 
---

### 창작자(크리에이터)

- 크리에이터는 nft를 판매 등록하는 시점에 signer로서 EIP-712를 통해 signature를 생성한다

- signer._signTypedData(domain, types, voucher) 를 통해서 크리에이터의 서명을 return 받는다

- 이렇게 생성된 byte형식의 signature는 구매자가 구매할때 호출되어 redeem 함수에서 실행된다.

- 구매가 redeem 되기 위해서 voucher의 signature를 통해서 창작자의 주소를 확인하고 나머지 정보들로 nft를 minting 할 때 사용한다.(tokenId, minPrice, uri)

---

### 구매자

- 구매 시에 smart contract의 redeem 함수가 실행된다.

- _verify(voucher)를 통해서 창작자(크리에이터)의 address를 알아낸다.

- _verify의 _hash(voucher)를 이용하여 typedData를 생성하고 ECDSA 알고리즘을 통해 창작자(크리에이터)의 address를 복구한다.

---

### 요약

- EIP-712를 이용하여 창작자(크리에이터)는 nft의 tokenId, uri, minPrice, signature를 이용해서 struct 형식의 voucher를 만든다.
- minPrice가 기입되었기 때문에 자동적으로 판매 등록이 됩니다.
- 구매자가 판매 등록 listing 되어 있는 nft를 사게 되면 off-chain 상에 저장되어 있는 해당 nft의 voucher를 불러옵니다.
- voucher의 signature를 통해서 창작자(크리에이터)의 address를 복구하고 tokenId, uri를 이용해서 nft를 minting합니다.
- minting이 진행되고 나서 transfer를 통해 창작자(크리에이터)에서 구매자로 nft를 이전합니다.