---
title:  "[BlockChain & Vue]Lazy Minting에 대하여(Part2)"
excerpt: "About Lazy Minting(Part2)..."

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2023-01-02
---

## Lazy Minting(Part2)
<br />

### Signature
> Signature : wallet만을 이용해서 특정 EIP712에 해당하는 서명을 도출해냄
- 온체인(blockchain / on-chain)과 상호작용이 없이 순전히 오프체인(off-chain)을 이용하여 만듬
- Signature 생성시 domain에 보증할 EIP smart contract 정보(domain), voucher의 실제 값(voucher), voucher의 구조체(types)를 사용 
- 만들어진 서명(signature)은 온체인(on-chain)이 아닌 오프체인(off-chain)인 데이터베이스(database)에 저장함

---

### Domain
> Domain : 서명이 유효한 contract 및 제반 사항을 확인함 
- name : EIP712 smart contract의 constructor name
- version : EIP712 smart contract의 constructor version
- chainId : wallet이 최근 선택한 network chain의 고유 id
- vertifyContract : EIP712 smart contract의 contract address

---

### Wallet address & Contract address -> Signature(off-chain)
> wallet을 이용하여 signature 생성
- 만들어진 서명(signature)은 데이터베이스(database)에 저장함
- contract address + user wallet address -> Keccak256 Hash -> 332 entry Uint8Array -> Signature
<br />

#### Javascript(off-chain)

```javascript
import Web3 from 'web3';
import { BigNumber, ethers} from 'ethers'

const SIGNING_DOMAIN_NAME = "EIP712"
const SIGNING_DOMAIN_VERSION = "1"

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

<br />

---

### Signature -> Wallet address(on-chain)
> EIP smart contract를 이용하여 address 도출
- **_ECDSA.sol_** openzeppelin library 사용
- smart contract(on-chain)을 이용하여 signature로 wallet address 도출
- signature -> Construct Keccak256 Hash -> toEthSignedMessageHash -> Recover the signer's wallet address(public key)


> _hash(voucher)
> - _hashTypedDataV4를 이용하여 voucher parameter를 EIP712 메세지의 인코딩된 해시를 반환

> verify(voucher, signature)
> - ECDSA.recover(_hash(voucer), signature)를 통해서 signer의 wallet address를 도출해냄

>  redeem(redeemer, voucher)
> - verify 함수와 voucher parameter를 이용하여 signer의 address를 도출함
> - 도출된 signer의 address와 voucher의 nft 정보를 이용해서 mint함 
> - mint 후 transfer(redeemer, voucher.tokenId)를 이용하여 nft 소유권을 구매자에게 양도

> **민팅 & 구매 시 : minting transaction + transfer transaction 두 번 발생(가스비 2번 발생)**<br/>
> **레이지민팅 & 구매 : minting, transfer transaction 한 번 발생(가스비 1번 발생)**<br/>
> **가스비 : 레이지민팅 << 민팅**


```javascript
const redeemer = (await this.getEthWallet())[0];
const mynftContractAddress = new web3.eth.Contract(nftContract.abi, nftContractAddress);
const signer = '0x0000000000000000000000000000000000000000'
const nonce = await web3.eth.getTransactionCount(
  publicAddress,
  "latest"
); 
const tx = {
  from: redeemer,
  to: nftContractAddress,
  nonce: nonce,
  value: ethers.utils.parseEther(price.toString()).toHexString(),
  data: mynftContractAddress.methods.redeem(redeemer, voucher).encodeABI(),
};
```

#### Smart Contract(on-chain)

```solidity
//SPDX-License-Identifier: Unlicense
pragma solidity ^0.8.7;
pragma experimental ABIEncoderV2;

import "@openzeppelin/contracts/access/AccessControl.sol";
import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "@openzeppelin/contracts/utils/cryptography/ECDSA.sol";
import "@openzeppelin/contracts/utils/cryptography/draft-EIP712.sol";

contract nftToken is ERC721URIStorage, EIP712, AccessControl {
  using ECDSA for bytes32;

  bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

  mapping (address => uint256) pendingWithdrawals;

  constructor(address payable minter)
    ERC721("erc721", "my-nft") 
    EIP712("EIP712", "1") {
      _setupRole(MINTER_ROLE, minter);
    }


  struct NFTVoucher {

    uint256 tokenId;
    uint256 minPrice;
    string uri;
    bytes signature;
  }

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

  function _verify(NFTVoucher calldata voucher, bytes memory signature) internal view returns (address) {
    bytes32 digest = _hash(voucher);
    return ECDSA.recover(digest, signature);
  }

  function supportsInterface(bytes4 interfaceId) public view virtual override (AccessControl, ERC721) returns (bool) {
    return ERC721.supportsInterface(interfaceId) || AccessControl.supportsInterface(interfaceId);
  }
}
```


---

