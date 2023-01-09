---
title:  "[BlockChain]ERC1155"
excerpt: "ERC1155 & ERC721 & ERC20"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2023-01-09
---


## ERC-1155(Ethereum Request for Comments - 1155)이란?
<br />

- ERC-1155는 기존에 존재하던 두가지 주요 토큰 표준  smart contract interface(ERC-20, ERC-721)의 성질을 동시에 가지는 samrt contract interface(ERC-1155) 
- ERC-1155는 대체 가능한 토큰 ERC-20, 대체 불가능한 토큰 ERC721의 두가지 성질을 모두 가지는 개선된 이더리움 표준안
- Atomic Swap 기술을 사용하여 적은 트랜잭션을 통한 자산 교환이 가능
- ERC-1155를 1개 발행하면 ERC-721의 성격, ERC-1155를 여러 개의 컬렉션으로 발행하면 ERC-20의 성격
- ERC-721은 1개의 고유한 성질을 가진 NFT를 발행하는데 사용이 되며, ERC-20은 WETH, WKLAY와 같은 동일한 성격을 가지는 토큰을 발행하는데 사용

<br/>

---

### ERC-721, ERC-20, ERC-1155

<br/>

- ERC-721 : 한 번의 트랜잭션으로 하나의 대체 불가능한 토큰에 대하여 하나의 이벤트만 발생 가능
- ERC-20 : 한 번의 트랜잭션으로 여러 개의 대체 가능한 토큰에 대하여 하나의 이벤트만 발생 가능
- ERC-1155 : 한 번의 트랜잭션으로 여러 개의 대체가능/반 대체 가능/대체 불가능 토큰에 대하여 하나의 이벤트 발생 가능

<br/>

![image info](/assets/img/erc.png)
<img src="/assets/img/erc.png" alt="" width="0" height="0">

<br/>

---

### ERC-1155의 장점

<br/>

- 가스 절약 : 단일 트랜잭션으로 일괄 전송, 일괄 민팅이 가능해서 가스 비용이 크게 절감됨
- 고급 기능 : 토큰 burn 부터 토큰 upgrade까지 더 많은 기능 사용 가능
- 다중 일괄 트랜잭션 : ERC-1155 토큰을 이용하여 단일 트랜잭션에서 여러 토큰 ID를 보낼 수 있음

<br/>

---

### ERC-1155 smart contract(Basic)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;

import "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/token/ERC1155/extensions/ERC1155Burnable.sol";
import "@openzeppelin/contracts/token/ERC1155/extensions/ERC1155URIStorage.sol";
import "@openzeppelin/contracts/token/ERC1155/extensions/ERC1155Supply.sol";

contract ERC1155token is ERC1155, Ownable, ERC1155Burnable {
    
  mapping(uint => string) public tokenURI;

  constructor() ERC1155token(" <write your json uri> ") {
  }
 
  function mint(uint _id, uint _amount) external onlyOwner {
    _mint(msg.sender, _id, _amount, "");
  }

  function mintBatch(address _to, uint[] memory _ids, uint[] memory _amounts) external onlyOwner {
    _mintBatch(_to, _ids, _amounts, "");
  }

  function burn(uint _id, uint _amount) external {
    _burn(msg.sender, _id, _amount);
  }

  function burnBatch(uint[] memory _ids, uint[] memory _amounts) external {
    _burnBatch(msg.sender, _ids, _amounts);
  }

  function burnForMint(address _from, uint[] memory _burnIds, uint[] memory _burnAmounts, uint[] memory _mintIds, uint[] memory _mintAmounts) external onlyOwner {
    _burnBatch(_from, _burnIds, _burnAmounts);
    _mintBatch(_from, _mintIds, _mintAmounts, "");
  }

  function setURI(uint _id, string memory _uri) external onlyOwner {
    tokenURI[_id] = _uri;
    emit URI(_uri, _id);
  }

  function uri(uint _id) public override view returns (string memory) {
    return tokenURI[_id];
  }

  //safeBatchTransferFrom(address from, address to, uint256[] ids, uint256[] amounts, bytes data)
  //totalSupply(uint256 id)
  //balanceOf(address account, uint256 id)
  //exists(uint256 id)

}
```
---

### ERC-115 & Gas Fee
 
> **최근 트랜드에 따라서 ERC-721을 사용하던 옛날의 NFT 마켓과 다르게 요새는 ERC-721(non-fungible)과 ERC-1155(semi-fungible)를 이용하며 발행할 NFT 개수를 정하고 Lazy Minting을 사용하여 사용자의 가스비 사용을 최소화 시킴**

> **Opensea에서는 또한 Seaport protocol을 이용하여 사용자의 가스비를 최대한 절감시키고 있음**