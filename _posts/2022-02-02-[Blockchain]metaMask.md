---
title:  "[Blockchain]MetaMask api에 대하여"
excerpt: "About MetaMask..."

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2022-01-18
---


## MetaMask

- 소개 
    - MetaMask는 웹 브라우저에 삽입되어, 여러 이더리움 테스트넷을 비롯하여 이더리움 메인넷에서 직접 가동되는 지갑입니다.
    - MetaMask는 중앙집중형 웹과 탈중앙형 이더리움 블록체인 간의 가교 역할을 담당합니다.
    - MetaMask는 Infura라는 이더리움 노드와 연결하여 Proxy를 통해 스마트 컨트랙트를 가동하도록 해 주어야 합니다.
    - MetaMask의 핵심은 Ethereum, private key, transaction입니다.

- 순서
    - npm install web3 
    - Metamask 설치
    - transaction logic 및 API 구현
---

## Priavte Key
private key 관리
- 이더/토큰을 누군가의 송금을 하거나 스마트 컨트랙트의 특정 함수 실행을 위해서는 sendRawTransaction을 하기 전에 data signing을 해야한다.
signing을 위해서 priavte key의 정보가 필요한데 chrome localstorage에서 이 값을 관리

- 니모닉 코드 추출 하기
```javascript
chrome.storage.local.get('data',result => {
    var vault = result.data.KeyringController.vault
    console.log(vault)
})
```
JSON형식으로 된 데이터가 추출된다(과거에는 WINDOW LOCAL STORAGE에 저장되어있었음)

추출된 데이터는 VAULT DATA이고 VAULT DATA를 통해 METAMASK 로그인시 입력한 패스워드와 결합하여 특정 decryptor 소스를 통해 니모닉 코드 추출

> https://metamask.github.io/vault-decryptor/

> https://github.com/nujabes403/metamaskVaultDecrypt

위의 사이트는 metamask vault를 해독할 수 있는 javascript source
password, vault data를 통해 decryptor
metamask가 내 local에 존재하는 privatekey를 관리하는 방법
---
ehter, transaction history 상태가 어떻게 반영이 되는지

네트워크 스냅 샷(network 탭에 존재)
background 동작으로 일정시간 같은 패턴으로 네트워크에 쿼리를 날림

metamask는 이더리움 네트워크에 접속하기 위해서 클라우드 서비스인 infura api 서비스를 사용
20초 간격으로 반복적 패턴으로 현재 block번호, block번호를 통한 현재 블록의 정보를 가져옴

1. https://api.infura.io/v1/ticker/ethusd (이더리움 USD 가격정보 )
2. https://api.infura.io/v1/status/metamask (이더리움 네트워크 상태체크)
3. https://api.infura.io/v2/blacklist ( MetaMask에서 관리하는 블랙리스트 )

이더의 USD TRICKER 정보, METAMASK에서 각 이더리움 네트워크의 상태 체크 정보, 주기적으로 http request

접속시
1. https://api.infura.io/v1/jsonrpc/rinkeby/eth_getTransactionCount?params=%5B%220x8bbd95a66902a0dad03d1850926ab062d9202fb6%22%2C%220x3858a0%22%5D ( 내주소의 트랜잭션 카운트 구하기 )
새로운 트랜잭션 생성시 nonce를 구하기 위해 지속적으로 확인
2. https://api.infura.io/v1/jsonrpc/rinkeby/eth_getBalance?params=%5B%220x8bbd95a66902a0dad03d1850926ab062d9202fb6%22%2C%220x3858a1%22%5D ( 접속 네트워크 지갑에 들어있는 이더리움 밸러스 체크 )


---
송금 transaction
1. nonce 구하기(transaction에 필요한 nonce를 만들기 위해 th_getTransactionCount를 수행)
2. 네트워크가 아닌 메타마스크 프로그램 내부에서 signedTransaction 진행
3. sendRawTransaction을 통해 서명된 transaction raw data 전송
4. transaction 처리가 완료될 때까지 transaction hash값을 eth_getTransactionByBash로 계속 보냄
5. 블록에 트랙잭션 해시가 포함되고 처리가 됨
6. 현재 지갑의 이더 잔고를 확인(eth_getBalance)
7. 이더 트랜잭션 receipt를 호출(eth_getTransactionReceipt)
8. metamask transaction history는 websql이나 별도의 storage 활용

주의) sendRawTransaction시 온라인 상태가 아닌 오프라인 상태에서 transaction 데이터를 private key로 signing

transaction data signing -> via Web3 library


---
web3.js
> https://web3js.readthedocs.io/en/v1.2.0/web3-eth.html#signtransaction

transaction data signing 하기

web3 객체 생성시 기본적으로 접속해 있는 사람의 계정은 unlock 상태여야 한다.

transaction data : 송금 data
web3.eth.signTransaction API 로 rawTx를 만든다.(JSON 구조의 데이터)

> const signTransacitonData = await web3.eth.signTransaction(JSON.parse(this.transactionData))

web3.eth.signTransaction을 통해 해당 transaction data를 JSON parse를 하여 input에 넣어 주고 rawTx를 뽑아낸다

```javascript
const jsonData = JSON.stringify({
     "jsonrpc" : "2.0",
     "method" : "eth_sendRawTransaction",
     "params" :[this.rawTransactionData],
     "id" : "1"
})
const result = $.ajax({
  url :
    'https://rinkeby.infura.io/v3/a0790b14a0164dd7b6c3d6c28d72eb8e',
  async : false,
  type : 'POST',
  data : jsonData,
  dataType : 'application/json'
})
console.log(result.responseText)
```
sendRawTransaction의 경우 web3 API Spec으로 존재하지만 MetaMask에서 Infura에 eth_sendRawTransaction을 JSON-RPC의 Spec으로 처리한 것과 같이 처리   

JSON-RPC SPEC
> https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_sendrawtransaction

sendRawTransaction으로 Infura를 통해서 Rinkeby network transaction을 요청하게 되면 transactionHash값이 정상적으로 출력됨

한번 철된 rawTx를 재사용하면 안된다. rawTX에는 Transaction 발생 주체인 계정의 nonce 값이 포함되기 때문에 nonce로는 처리가 불가능하기 때문

metamask를 통해 계정 A에서 계정 B로 처리되는 과정에서 계졍의 transaction count의 nonce를 구해서 보내고자 하는 eth의 양과 누구로부터 누구에게 보낼것인지에 대한 전반적인 transaction data를 signedTransactoin이라는 web3 spec을 통해 오프라인으로 만들어준다
만들어준 rawTx를 JSON-RPC Spec에 존재하는 sendRawTransaction으로 send 하고 바로 transactionHash 값이 출력됨
metamask는 이 transaction이블록에 포함 될때까지 transaction을 주기적으로 check

블록 생성순간 현재 사용자의 ether 잔고를 확인, pending되어 있던 transaction을 승인 완료로 처리, 

metamask 는 송금 transaction을 처리한다

smart contract 생성 후 sendRawTransaction을 통해 배포 

1. contraction를 remix에서 byteCode로 전환
2. signedTransaction으로 실행하면 rawTx 생성
3. raw값을 복사해서 sendRawTransaction을 실행
4. etherscan에서 확인
> https://rinkeby.etherscan.io/address/0xd756ed39480ef70f315454de0d5a4a1aac2e7d36

5. contract가 정상적으로 creation된 것을 확인
6. contract address 복사후 remix에서 At Address로 객체 생성
7. At Address로 rinkeby에 배포된 contract 주소를 넣어 생성

송금이 아닌 contract 특정 function 실행을 위해서 metamask hexdata 필드를 활성화 해야됨
web3 library spec 중 encodeFunctionCall 스팩을 이용해서 function parameter를 직렬화시켜 bytecode생성
bytecode hex데이터 input에 입력, 받는 주소에 contract 주소 입력시 전송 완료!







`출처 : https://medium.com/day34/metamask-%EC%A7%80%EA%B0%91-%EB%9C%AF%EC%96%B4%EB%B3%B4%EA%B8%B0-650ba9920f6a`


---
1. frontend(web3) -> metamask (request to sign transaction)
2. metamask -> user (confirm)
3. user -> metamask(yes)
4. metamask -> smart contract(send signed transaction)


