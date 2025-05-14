---
title: " Nginx Usage Of Proxy Header, GeoIP2 Redirection With Configuration"
excerpt: "[Web Server] About Nginx Usage Of Proxy Header, GeoIP2 Redirection With Configuration"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2025-05-12
---


### 들어가면서
  - Issue
    - 현재 프로젝트에서 Web Server로 nginx를 사용하고 있습니다. 개발 건 중에 유렵, 중국 등 해외 특정 법인에서 접근 시 같은 도메인이지만 접근되는 페이지가 다르거나 같은 페이지에서도 해당 기능을 사용 및 다른 기능을 사용하도록 해야 하는 부분이 있었습니다
  - Consideration
    - 다른 도메인으로 처리하는 것이 아닌 같은 도메인 상의 접근이 달라지는 부분을 고려

  - Solution 1(GeoIP2)
    - nginx geoip2를 이용해서 china europe 지사의 ip를 파악
    - 특정 페이지 및 기능 제어 : nginx proxy header 전달을 통한 변수 제어
    - 한계 : geoip2 module은 db를 주기적으로 update를 해줘야함 -> 유료화 가능성과 데이터의 주기적인 변경에 대응하기 내부 보안 환경이 적합하지 않음

  - Solution 2(CIDR)
    - nginx IP 대역(CIDR)를 이용한 접근하는 지사 나라 및 도시 파악
    - 한계 : GeoIP2와 같은 한계
    - nginx에서 접근 ip를 고려하여 redirect or proxy header 설정을 통해 백엔드/프론트에서 param 전달


  - **Solution 3(Specified Path) -> Final Conclusion**
    - 해외 지사에는 /global 같은 특정 도메인을 제공하여 요청한 방식대로 redirect 및 특정 기능을 제한한다
    - nginx를 통해 /global 입력 시 ?path=global로 redirect하여 parameter를 통해서 특정 기능을 통제하거나 nginx proxy header를 넘겨주는 방식으로 변수를 제어한다
    - 아래의 nginx에서 제공해주는 option 및 component의 적절한 조합을 통해서 제한된 상황에서의 Issue를 해결할 수 있었습니다

<br />

---

> ### Proxy Options


  ```java
  location/ {
    proxy_pass http://localhost:8080;
    proxy_http_version  1.1;

    proxy_cache_bypass                 $http_upgrade;
    proxy_set_header Upgrade           $http_upgrade;
    proxy_set_header Connection        "upgrade";
    proxy_set_header Host              $host;
    proxy_set_header X-Real-IP         $remote_addr;
    proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-Host  $host;
    proxy_set_header X-Forwarded-Port  $server_port;
  }
  ```

  - #### proxy_pass http://localhost:8080
    - Client의 요청을 내부 localhost:8080 서버로 전달
      ```http
      GET /api/select/user → Nginx → localhost:8080/api/select/user
      ```

  - #### proxy_http_version  1.1;
    - Http 버전을 1.1로 설정(Web Socket, Keep-alive.. 최신 기능 지원을 위한 설정)
  
  - #### proxy_cache_bypass $http_upgrade;
    - Web Socket upgrade 요청 시 캐시를 무시하도록 설정($http_upgrade 값이 있는 경우 프록시 캐시 우회를 강제)
    - 캐시된 응답이 아닌 실시간 서버 연결을 원할 경우 캐시 무효화
  
  - #### proxy_set_header Upgrade $http_upgrade; & proxy_set_header Connection "upgrade";
    - Web Socket 연결을 위해 필수적인 Header 설정
    - Http 연결을 WebSocket으로 Upgrade할 수 있도록 명시
    - Nginx가 해당 Header를 전달하지 않으면 WAS는 Web Socket 연결을 거부

      ```http
      GET /myService/chat HTTP/1.1
      Upgrade: websocket
      Connection: Upgrade
      ```
  
  
  - #### proxy_set_header Host $host;
    - 원본 요청의 Host 값을 WAS에 전달
    - WAS에서 Host를 통해서 분기 처리가 가능해진다
      ```http
      Host : woon7650.com
      ```
  
  - #### proxy_set_header X-Real-IP $remote_addr;
    - 실제 Client의 IP를 WAS에 전달
    - Log, IP 관련 보안 설정에 사용된다
      ```java
      String userIP = StringUtil.checkNull(request.getHeader("X-Real-IP"));
      ```
  
  - #### proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    - Proxy Chain에 따라 전달된 Client IP List
    - Client -> Proxy A -> Proxy B -> Proxy C -> WAS 인 경우 Client와 WAS 사이의 중간 지점의 IP가 리스트로 전달됨
      ```http
      X-Forwarded-For : 111.111.111.111 222.222.222.222 333.333.333.333
      ```
  
  - #### proxy_set_header X-Forwarded-Proto $scheme;
    - 해당 Request가 Http인지 Https인지 WAS에 알려준다
      ```java
      String requestType = StringUtil.checkNull(request.getHeader("X-Forwarded-Proto"));
      if("http".equals(requestType))
      ```
  
  - #### proxy_set_header X-Forwarded-Host $host & proxy_set_header X-Forwarded-Port $server_port;
    - Request에 사용된 Host와 Port를 WAS에 전달한다
    - Header로 부터 host와 port를 통해서 정확한 request url을 얻을 수 있다다
      ```java
      String host = StringUtil.checkNull(request.getHeader("X-Forwarded-Host"));
      String port = StringUtil.checkNull(request.getHeader("X-Forwarded-Host"));
      ```

<br />

---


- ### default.conf

  - #### Nginx Basic default.conf

    - nginx.conf에서 include해서 사용
    - 서버 설정 관련 파일이며 default2.conf, default3.conf.. 여러 개의 파일을 추가해서 서버 관련 설정을 추가로 포함시킬 수 있다
    
      ```java
      server {
        listen       80;
        server_namve  localhost;
        root   /usr/share/nginx/html;
        index  index.html index.htm;
        error_page 404              /404.html;

        location = /404.html {
            root   /usr/share/nginx/html;
        }

        error_page 500 502 503 504  /50x.html;

        location = /50x.html {
            root   /usr/share/nginx/html;
        }

        location / {
            try_files $uri $uri/ =404;
        }
      }
      ```
  
  - #### Components
    - listen
      - 해당 port로 들어오는 요청을 해당 server {} 내용에 맞게 처리한다
    - server_name
      - host 이름을 지정하고 가상 호스트가 있는 경우 해당 host 이름을 명시하면 된다
      - 로컬에서 작업하고 있는 내용을 nginx를 통해 띄우려고 하는 경우에는 localhost라고 명시하면 된다
    - error_page
      - 요청 결과의 http 상태코드가 지정된 http 상태코드와 일치할 경우  해당 url로 이동한다
      - 400번대, 500번대 등의 에러처리를 위해 사용한다
    - location /
      - 처음 request가 들어왔을 때 보여줄 화면이 속해 있는 경로와 초기 페이지인 index를 지정해준다
      - server_name이 127.0.0.1인 경우 : 127.0.0.1로 요청이 들어왔을 경우 index.html, index.htm로 정의된 파일을 보여준다
    - ssl_certificate
      - 생성된 인증서 설정 경로
    - upstream
      - proxy_pass 지시자를 활용해 nginx가 받은 request를를 넘겨 줄 서버를 정의

- ### nginx.conf

  - #### Nginx Basic nginx.conf
    - nginx의 기본 설정 담당

      ```java
      user nginx;
      worker_processes auto;

      error_log /var/log/nginx/error.log warn;
      pid /var/run/nginx.pid;

      events {
          worker_connections 1024;
      }

      http {}
        include       /etc/nginx/mime.types;
        default_type  application/octet-stream;

        log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                        '$status $body_bytes_sent "$http_referer" '
                        '"$http_user_agent" "$http_x_forwarded_for"';

        access_log  /var/log/nginx/access.log  main;

        sendfile        on;
        #tcp_nopush     on;

        keepalive_timeout  65;
        #gzip  on;

        include /etc/nginx/conf.d/*.conf;
      }

      ```

  - #### Components
    - worker_processes
      - worker_process로 유입되는 이벤트를 처리하는 프로세스 수
      - 보통 cpu 코어만큼 추천
    - error_log
      - nginx의 에러 로그가 쌓이는 경로
    - pid
      - nginx의 프로세스 아이디 저장 경로
    - worker_connections
      - worker process가 동시에 처리할 수 있는 접속자 수 
      - default : 1024
    - include
      - 포함시킬 외부 파일
    - default_type
      - 웹 서버의 기본 content type 정의
    - log_format
      - 로그 형식을 지정
    - access_log 
      - 접속 로그가 쌓이는 경로
    - sendfile
      - sendfile() 함수의 사용 여부
    - keepalive_timeout
      - 클라이언트에서 연결이 유지될 시간을 정의함
      - default : 65



<br />

---