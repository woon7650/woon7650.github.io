---
title: "JWT(Json Web Token)"
excerpt: "[Spring Security] JWT "

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2024-05-21
---

#### 0. JWT(Json Web Token)

> JSON Web Token (JWT) is an open standard (RFC 7519) that defines a compact and self-contained way for securely transmitting information between parties as a JSON object. This information can be verified and trusted because it is digitally signed. JWTs can be signed using a secret (with the HMAC algorithm) or a public/private key pair using RSA or ECDSA.

> JSON Web Token(JWT)은 당사자 간의 정보를 안전하게 전송하기 위한 간결하고 자체적인 방법을 JSON 객체로 정의하는 개방형 표준(RFC 7519)입니다. 이 정보는 디지털 방식으로 서명되기 때문에 검증되고 신뢰될 수 있습니다. JWT는 비밀(HMAC 알고리즘으로)을 사용하거나 RSA 또는 ECDSA를 사용하여 공개/개인 키 쌍을 사용하여 서명할 수 있습니다.

- 서버에서 클라이언트 인증을 확인하는 방식은 Cookie, Session, Token 3가지 방식이 있습니다.
- 이번 글에서는 우선 Token 방식의 인증 방식(Authentication Method)에 대해서 다룰 생각입니다.

<br />

---

### 1. JWT(Json Web Token)

- #### 1.1 Conception

  - 클라이언트와 서버간에 인증에 필요한 정보들을 암호화시킨 JSON 토큰이다.
  - JWT 토큰(Access Token)을 Http header에 실어 서버에서 클라이언트를 식별한다.
  - 해당 토큰은 클라이언트에 대한 정보를 인코딩하여 가지고 있다.

- #### 1.2 Json Format

  - JWT(Json Web Token) = Header + Payload + Signature
    ![image info](/assets/img/jsonFormat.png)
    <img src="/assets/img/jsonFormat.png" alt="" width="0" height="0">

  - ##### 1.2.1 Header
    - alg, typ에 대한 정보를 가지고 있다.
    - alg은 서명 암호화 알고리즘을 결정한다.
      - HMAC, SHA256, RSA
    - typ은 토큰 유형을 결정한다.

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

- ##### 1.2.2 Paylod

  - Base64URL 방식으로 인코딩 되어있다.
  - 하나의 클레임(Claim)은 이름-값(name-value)를 한 쌍으로 구성된다.

  - ###### 1.2.2.1 Claim
    - Registed Claims(등록된 클레임) : 미리 정의된 클레임(Claim)
      - sub(subject) : 제목
      - iss(issuer) : 발급자
      - aud(audience) : 대상자
      - exp(expiration) : 만료 시간
      - nbf(not before) : 활성화 날짜
      - iat(issued at) : 발급 시간
      - jti(jwt id) : 고유 식별자
    - Public Claims(공개 클레임) : 사용자가 정의할 수 있는 공개용 클레임(Claim)
    - Private Claims(비공개 클레임) : 사용자의 특정할 수 있는 정보를 담은 클레임(Claim)

```json
{
  "exp": "1535300000000", //Registed Claim
  "woon7650.github.io": true, //Public Claim
  "name": "woon7650" //Prvate Claim
}
```

- ##### 1.2.3 Signature
  - Header와 Payload의 데이터 무결성 및 변조 방지를 위한 서명 정보.
  - 서버에서 Signature를 비교해서 위조된 토큰인지 아닌지 여부를 판단할 수 있다.

```json
HMACSHA256(
    Base64Url(Header) + "." +
    Base64Url(PayLoad),
    server's key
)
```

<br />

- #### 1.3 Example

  - **https://jwt.io/** 에서 실제 JWT(Json Web Token)의 인코딩(Encoding), 디코딩(Decoding) 할 수 있다.

![image info](/assets/img/jsonExample.png)
<img src="/assets/img/jsonExample.png" alt="" width="0" height="0">

<br />

---

### 2. Access Token, Refresh Token

- #### 2.1 신뢰성(Realiablity)

  - 사용자 JWT : A(Header) + B(Payloaod) + C(Signature)
  - Payload가 변경됬을 때
    - 변경된 JWT : A + B' + C
    - 서버에서 검증 후 JWT : A + B' + C'
    - 서명(Signature) 불일치 -> 유저의 정보가 임의로 변경됨을 알 수 있다.

- #### 2.2 주의사항(Caution)

  - JWT는 Base64로 암호화하기 때문에 복호화도 가능하다.
  - Payload 부분이 노출될 수 있기 때문에 중요한 정보는 삼가해야 한다.
  - 토큰 인증의 목적은 정보 보호가 아닌 **위조 방지**이다.

- #### 2.3 해결책(Solution)

  - 현업에서는 토큰 탈취의 위험성이 있기 때문에 Access Token, Refresh Token으로 이중으로 나누어 인증한다.
  - Access Token(접근용 토큰), Refresh Token(재발급용 토큰)으로 나뉜다.

  - ##### 2.3.1 Access Token

    - 클라이언트가 갖고 있는 유의 정보가 담긴 토큰이다.
    - 서버에서 해당 토큰에 있는 정보를 바탕으로 응답한다.

  - ##### 2.3.2 Refresh Token
    - 새로운 Access Token을 발급해주기 위한 토큰이다.
    - DB에 유저 정보와 같이 기록한다.

<br />

---

### Refernce

- https://jwt.io/introduction
- https://velog.io/@chuu1019/%EC%95%8C%EA%B3%A0-%EC%93%B0%EC%9E%90-JWTJson-Web-Token
- https://inpa.tistory.com/entry/WEB-%F0%9F%93%9A-JWTjson-web-token-%EB%9E%80-%F0%9F%92%AF-%EC%A0%95%EB%A6%AC#jwt_json_web_token_%EC%9D%B4%EB%9E%80
- https://datatracker.ietf.org/doc/html/rfc7519#section-4.1
