---
title:  "CSRF(Cross-site request forgery) Attack"
excerpt: "CSRF Attack"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2022-01-14
---

## Cross-site request forgery(CSRF)
- 개념
  - 사이트 간 요청 위조 요청을 통해 공격하는 기법입니다.
  - 사용자 의지와 무관하게 공격자의 의도대로 사용자의 권한을 통해서 서버에 특정 요청을 하도록 합니다.
  - 사용자가 사이트에 정상적으로 로그인시 링크를 클릭하도록 피싱합니다. 링크를 클릭시에 이미 로그인 되어있던 사이트의 사용자의 패스워드를 변경합니다.(browser tab을 여러 개 사용 시 취약합니다.)
- 공격 과정
  - 공격자는 CSRF `<script>`가 포함된 URL을 게시물/이메일을 통해 피싱합니다.
  - 사용자가 다른 웹사이트에 로그인을 한 상태로 CSRF `<script>`가 포함된 게시물을 열람합니다.
  - CSRF `<script>`가 포함된 게시물이 응답하여 CSRF `<script>`가 실행이되어 사용자 권한을 빼앗아 갑니다.
- 조건
  - 사용자가 웹사이트에 로그인을 한 상태에서 CSRF `<script>`가 포함된 페이지를 열 때 발생합니다.
- CSRF 실제 발생 예시

    > 2008년도 옥션 해킹 사고의 원인(CSRF 공격)
    > - 공격자가 옥션 운영자에게 CSRF 코드가 포함된 이메일을 보냈습니다.
    > - *`<img src="http://auction.com/changeUserAcoount?id=admin&password=admin" width="0" height="0">`*
    > - 관리자는 이미지 크기가 0이므로 전혀 알아차리지 못하고 URL이 열린다.
    > - 공격자는 관리자 권한을 가지게 됩니다.

- 보완 방법
  - **Referer 검증**
    - Back
      > request header에 있는 referer 속성을 검증합니다.(요청을 한 페이지의 정보 -> 정보가 없다면 접속 차단)

      ```java
      String referer = request.getHeader("REFERER");
      if( referer != null && referer.length() > 0){
      //접속 허용
      }else{
      //접속 차단
      }
      ```

    - Front
      > 다른 도메인명으로 접근하는 외부 접속을 차단합니다.(document.referer : 링크를 통해 현재 페이지로 이동 시킨, 전 페이지의 URI 정보를 반환)

      ```jsp
      var hostName = document.referer;
      if(hostName == '페이지 이동 전 페이지의 도메인'){
      //접속 허용
      }else{
      //접속 차단
      }
      ```

    - 핵심 : 현재 페이지 이전의 페이지 정보가 없거나, 이전 페이지와 도메인이 다르면 접속을 차단합니다.
    
  - **CSRF Token 사용**
    - Back
      > 먼저 페이지로 넘어가기 전에 controller 단에서 session에서 String type의 random id를 session에 저장한 후 front로 CSRF_TOKEN값을 넘겨줍니다. 

      ```java
      session.setAttribte("CSRF_TOKEN",UUID.randomUUID().toString());
      ```

    - Front
      > 페이지에서 action으로 인한 서버와의 통신 발생시 controller에서 넘겨준 csrf_token 값을 hidden input 형태로 담아 controller로 보내줍니다.

      ```jsp
      <input type = "hidden" name="_csrf" value="${CSRF_TOKEN}"/>
      ```

    - Back(Front로 넘어간 후)
      > controller에서 front에서 넘어온 _csrf 값과 session에 저장되어있던 CSRF_TOKEN 값을 비교합니다.

      ```java
      session.getAttribte("CSRF_TOKEN");
      ```

    - 핵심 : 사용자의 *session, page*에 들고 있던 *CSRF_TOKEN* 값을 통해 서버와의 통신 발생시 *CSRF_TOKEN*값 비교를 통해 방지합니다

  - **CAPTCHA 사용**
    - 로봇인지 아닌지 체크해서 사용자가 의도한 요청인지 아니면 트리거로 동작하는지 걸러낼 수 있습니다.
  
  - **Spring Security**
    - pom.xml
      > Spring Security 3.2.0이후 부터 CSRF 방어 기능을 제공합니다.

      ```xml
      <properties>
		    <java-version>1.8</java-version>
		    <org.springframework-version>3.2.9.RELEASE</org.springframework-version>
    		<org.aspectj-version>1.6.10</org.aspectj-version>
	    	<org.slf4j-version>1.6.6</org.slf4j-version>
	    </properties>
      ```

  - **Post Method**
    - GET 방식 보다는 Post 방식을 지향합니다. 
    
---

## XSS & CSRF ATTACK
- XSS는 사용자가 특정 사이트를 신뢰한다는 점을 공격하는 기법입니다.(Client에서 발생)
  - 목적 : *cookie & session 탈취 및 웹사이트 변조*
  
- CSRF는 특정 사이트가 사용자의 브라우저를 신뢰한다는 점을 공격하는 기법입니다.(Server에서 발생)
  - 목적 : *사용자의 권한 도용*

---

## 마무리를 하면서 느낀점
- 저번에 다룬 *XSS attack*을 다루면서 비슷한 웹 취약점 공격인 *CSRF attack*에 대해 다루어 보고 싶었습니다. 실제 업무에서 시스템에서 직접 *Burp Suite* 툴을 이용해보면서 *CSRF attack*을 실제 해보고 *CSRF attack*에 취약하다는 것을 인지하고 *CSRF token*을 이용해서 취약점을 대비하는 코드를 작성했습니다.