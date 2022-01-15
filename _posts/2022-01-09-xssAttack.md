---
title:  "크로스사이트 스크립트(취약점)에 대하여"
excerpt: "취약점 분석 및 보완 방법에 대하여"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2012-09-01
---

## Cross Site Scripting(XSS)
- 개념
    - client web browser를 공격하는 기법입니다
    - javaScript의 태그 등을 악의적인 script에 작성하여 다른 사용자들의 개인정보 및 세션쿠키를 탈취하고 웹페이지 변조 공격을 합니다
- 종류
    - Reflected XSS
    - Stored XSS
> `조회합니다. ->  조회<script>alert('xss 공격시작')</script>합니다.`
- 공격 과정
    - 공격자는 XSS 공격에 취약한 사이트를 찾아서 악의적인 `<script>`가 포함된 URL를 사용자에게 노출 시킵니다.(Reflected XSS)
    - 공격자는 게시판에 `<script>`를 삽입해서 저장한 후 사용자에게 게시글의 URL을 노출한후 사용자가 게시글 조회시 공격을합니다.(Stored XSS)
    - `<input>`안에 `<script>`태그를 작성하고 서버에 저장하게 되면 사용자가 해당 내용을 읽을 때 `<script>`가 실행되면서 명령이 실행됩니다.
- 공격시 유출되는 정보
    - cookie 정보 및 session 정보
    - 악성코드 다운로드(site redirect)
    - 관리자 권한 탈취
    - 페이지 변조 후 노출
- 보완 방법
    1. function을 이용한 parameter값 필터링(태그 필터링)
        ```ts
        function XSSCheck(str,level){
            if (level == undefined || level == 0) {
                str = str.replace(/\<|\>|\"|\'|\%|\;|\(|\)|\&|\+|\-/g,"");
            }else if (level != undefined && level == 1) {
                str = str.replace(/\</g, "&lt;");
                str = str.replace(/\>/g, "&gt;");
            }
            return str;
        }
        ```

    2. lucy-xss-servlet-filter 설정(설정한 url, parameter, prefix로 시작하는 parameter 필터링 제외 가능합니다.)
    
        web.xml
        ```xml
        <filter>
	        <filter-name>xssEscapeServletFilter</filter-name>
	        <filter-class>com.navercorp.lucy.security.xss.servletfilter.XssEscapeServletFilter</filter-class>
        </filter>
        <filter-mapping>
            <filter-name>xssEscapeServletFilter</filter-name>
            <url-pattern>/*</url-pattern>
        </filter-mapping>   
        ```
        *web.xml* 및 *lucy-xss-servlet-filter-rule.xml* 작성 필요
        > 참고 : https://github.com/naver/lucy-xss-servlet-filter
    
---
## 마무리를 하면서 느낀점
- 저번에 session, cookie에 대해 다루면서 이번 기회에 cookie, session 관련 취약점을 다뤄보자는 생각이 들었고 정보 탈취라는 부분에 처음 접해보면서 상당히 흥미로웠습니다. 다음 페이지에서는 CSRF 공격을 session을 이용해서 취약점을 해결해 볼 생각입니다. 