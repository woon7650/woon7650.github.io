---
title: "JWT Authentication"
excerpt: "[Spring] Access/Refresh Token Part 1"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2024-06-27
---

#### 0. Cookie, Session, Token(JWT) Authentication

- 인증 방식
  - 쿠키 인증 방식
    - 최초 로그인 시 쿠키에 저장한 사용자 인증 정보로 요청(stateless)
    - CSRF, XSS 공격에 취약
  - 세션 방식
    - 최초 로그인 시 받은 SessionID를 포함한 쿠키로 인증 요청(stateful)
    - 디바이스별 인증 관리 및 하나의 계정 공유 관리 가능, 비정상적인 접근시 세션 삭제
    - Scale-out 시 처리, 서버 부하 문제를 고려해야 함
    - 세션 관리 위치
      - 메모리 영역/하드디스크
      - 서버 DB
  - JWT 토큰 방식
    - 최초 로그인 시 받은 Token으로 인증 요청(stateless)

- 쿠키 방식은 보통 보안상 취약점이 많아서 세션 또는 JWT 방식을 주로 사용한다. 
- 디바이스별 관리, 계정 공유 관리와 같은 비즈니스적 기능이 필요할 때는 세션 방식을 필요하지 않을 때는 JWT 방식을 사용하는 것이 합리적이라고 생각된다.

<br />

---

### 1. JWT Authentication(Access Token, Refresh Token)

- #### 1.1 Problem
  - 순정 JWT 방식
    - 서버 DB를 전혀 사용하지 않은 Stateless한 방식(서버가 Token의 상태를 보관하고 있지 않음)
    - 서버는 토큰을 발급한 후에 토큰에 대한 제어를 잃음
    - 안전한 로그아웃 방식을 구현할 수 없음
    
  - JWT 탈취
    - JWT는 사용자의 신원이나 권한을 결정하는 정보를 담고 있는 토큰
    - 서버는 사용자와 탈취한 사람을 구분할 수 없음

- #### 1.2 Solution

  - JWT 방식 + DB 사용(약간의 Stateful)을 통해 구현
  - 유효기간이 다른 2개의 JWT가 필요함(Access Token, Refresh Token)

  - ##### 1.2.1 Access Token

    - API 통신 시 리소스에 접근하기 위한 목적의 JWT
    - Access Token을 검증하여 사용자의 정보를 확인함(로그인 요청)

  - ##### 1.2.2 Refresh Token

    - Stateless한 Access Token에 대하여 탈취에 대한 보안을 강화하기 위해 Access Token의 발급을 위한 목적의 JWT
    - Access Token의 유효기간이 짧기 때문에 만료 시 주기적인 재발급을 위하여 유효기간이 긴 Refresh Token을 이용함


- #### 1.3 Stateless

  - Stateless Token : 서버는 한 번 발급한 Token에 대한 제어권을 가지고 있지 않음(발급 후 서버에 저장하지 않음)
    - Access Token : 서버 DB에 Token을 저장하지 않음(Stateless JWT)
    - Refresh Token : 서버 DB에 Token을 저장함(Stateful JWT)

- #### 1.4 Duration

  ![image info](/assets/img/staticRefreshToken.png)
  <img src="/assets/img/staticRefreshToken.png" alt="" width="0" height="0">

  - Access Token : 유효 기간이 짧음(ex: Microsoft: 60일, Amazon: 1시간)
  - Refresh Token : 유효 기간이 길음(ex: Microsoft : 1년)
  - 탈취당할 리스크가 큰 Access Token의 만료 기간을 짧게 Refresh Token의 만료 기간은 길게 설정함
  - 서비스의 유형에 적절한 유효기간 설정이 필요

<br />

---

### 2. Process


  - #### 2.1 FlowChart

    ![image info](/assets/img/JWTProcess.png)
    <img src="/assets/img/JWTProcess.png" alt="" width="0" height="0">

    - 사용자 로그인(ID/PW)를 통해서 서버DB 조회를 통해 사용자 확인
    - 일치 시(로그인 성공 시) Refresh Token, Access Token을 발급하고 Refresh Token은 저장
    - API 요청마다 Access Token을 헤더에 담아서 요청
    - Access Token이 만료되었다는 응답을 받으면 클라이언트 측에서 Refresh Token을 헤더에 담아서 재요청
    - 서버는 Refresh Token로 사용자 권한을 확인하고 응답에 새로운 Access Token을 보냄
    - Refresh Token와 Access Token이 모두 만료되었다면 재로그인을 해야함


  - #### 2.2 Validation Check

    ![image info](/assets/img/JWTLogic.png)
    <img src="/assets/img/JWTLogic.png" alt="" width="0" height="0">

    - Access Token 만료 시 : Refresh Token을 통한 Access Token 재발급
    - Refresh Token 만료 시 : Access Token을 통한 Refresh Token 재발급
    - Access Token, Refresh Token 만료 시 : 재로그인

<br />

---


### 3. JWT Problem, Solution

- #### 3.1 Problem

  - 유효기간이 긴 Refresh Token가 탈취된 경우
  - Refresh Token 탈취로 만료시점에 Access Token 우선 발급받는 경우
  - 한 명의 사용자에 대해 여러 Refresh Token이 저장되는 경우

- #### 3.2 Solution

  - Refresh Token BlackList(Logout)
    - 만료되거나 무효화(로그아웃)된 Refresh Token을 데이터베이스에 저장함
    - Access Token 만료시 Refresh Token으로 서버 DB 블랙리스트 테이블 조회
      - 블랙리스트에서 조회될 경우 : Access Token 재발급
      - 블랙리스트에서 조회될 경우 : 로그아웃된 유저라는 것을 인지하고 재발급 거절 및 Access Token 삭제

  - Refresh Token Rotation(Re-request)
    - Refresh Token을 통해 Access Token 발급 시 Refresh Token도 새로 발급

  - Extra Security Action
    - IP주소나 기기 정보 같은 정보를 토큰에 포함시켜서 탈취된 토큰의 외부 접근을 방지
    - 한 명의 사용자에 대하여 2이상의 Refresh Token 저장 시 로그아웃 처리
    - Refresh Token이 저장되는 테이블의 PK를 유저 이메일이나 유저 아이디로 처리(사용자에 대해 하나의 Refresh Token을 존재하도록 강제)

<br />

---
### Refernce

- https://velog.io/@chchaeun/%EC%9D%B8%EC%A6%9D%EA%B3%BC
- https://velog.io/@chuu1019/Access-Token%EA%B3%BC-Refresh-Token%EC%9D%B4%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B4%EA%B3%A0-%EC%99%9C-%ED%95%84%EC%9A%94%ED%95%A0%EA%B9%8C
- https://hudi.blog/refresh-token/
- https://velog.io/@jkijki12/Jwt-Refresh-Token-%EC%A0%81%EC%9A%A9%EA%B8%B0
- https://skatpdnjs.tistory.com/60
- https://engineerinsight.tistory.com/232
- https://ksh-coding.tistory.com/113#google_vignette