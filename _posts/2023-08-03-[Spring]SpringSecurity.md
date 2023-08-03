---
title:  "Spring Security"
excerpt: "[Spring] About Spring Security Part1"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2023-08-03

---

## 0. 들어가면서


#### Spring Security ??

> Spring Security는 강력하고 사용자 정의가 매우 용이한 인증 및 액세스 제어 프레임워크입니다. 이는 Spring 기반 애플리케이션의 보안을 위한 사실상의 표준입니다. Spring Security는 Java 애플리케이션에 인증과 인증을 모두 제공하는 데 중점을 둔 프레임워크입니다.

#### Authentication / Authorization

- Principal & Credential pattern : username & password
- Authentication 제출 -> 검증 -> Authorization 부여  


#### 용어 설명

- Authentication : 사용자가 본인이 맞는지 확인하는 절차
- Authorization : 인증된 사용자가 요청한 자원에 접근 가능한지 결정하는 절차
- Principal : 요청한 자원에 접근하는 대상
- Credential : 자원에 접근하는 대상의 비밀번호

#### Flowchart

![image info](/assets/img/springsecurity.png)
<img src="/assets/img/springsecurity.png" alt="" width="0" height="0">


<br />

---

### 1. HTTP Request Receive

- Browser(Client)로 부터 request를 받음
- id, password를 기반으로 인증, 권한 부여를 진행
- Application Filter 중에 Authentication Filters에 가장 먼저 도달

<br />

---


### 2. AuthenticationFilter

- AuthenticationFilter가 인증, 권한 부여를 위한 과정을 먼저 거침
- 위의 과정을 마친 후 Dispatcher Servlet으로 요청을 넘김
- 

<br />

---

### 3. UsernamePasswordAuthenticationToken

- Authentication Interface의 구현체
- Principal-Credential pattern을 반영한 객체
- attemptAuthentication 메소드를 통해 username, password를 이용하여 Authentication 생성
- 모든 접근 주체는 Authentication을 생성하고 SecurityContext에 보관 및 사용

<br />

- AbstractAuthenticationToken implements Authentication
- UsernamePasswordAuthenticationToken extends AbstractAuthenticationToken

<br />

---

### 4. AuthenticationManager

- authenticate 메소드를 제공하는 interface
- Authentication 객체를 받아 인증하고 성공 시 Authentication 객체를 return

<br />

---
### 5. ProviderManager(implements AuthenticationManager)

- AuthenticationManger의 구현체
- Spring Security가 관리하는 Bean
- ProviderManager는 여러 AuthenticationProvider를 순회하면서 인증 처리가 가능한 AuthenticationProvider를 찾아서 인증 처리 과정 위임

<br />

---

### 6. AuthenticationProvider

- AuthenticationProviders 중에 실제 검증을 수행하는 AuthenticationProvider
- principal을 바탕으로 해당 유저의 정보를 조회
- UserDetailService 구현체를 사용하여 유저의 정보가 담겨있는 객체를 반환 

<br />

---



### 7. UserDetailsService

- DB에서 유저 정보를 불러와서 유저의 정보가 담겨있는 객체를 만듬

<br />

---




#### Reference
