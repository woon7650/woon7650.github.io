---
title: " Nginx Usage Of Proxy Header, GeoIP2 Redirection With Configuration"
excerpt: "[Web Server] About Nginx Usage Of Proxy Header, GeoIP2 Redirection With Configuration"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2025-04-24
---


### 들어가면서
  - Issue
    - 현재 프로젝트에서 Web Server로 nginx를 사용하고 있습니다. 개발 건 중에 유렵, 중국 등 해외 특정 법인에서 접근 시 같은 도메인이지만 접근되는 페이지가 다르거나 같은 페이지에서도 해당 기능을 사용 및 다른 기능을 사용하도록 해야 하는 부분이 있었습니다. 
  - Solution
    - 다른 도메인으로 처리하는 것이 아닌 같은 도메인 상의 접근이 달라지는 부분을 고려
    - nginx에서 접근 ip를 고려하여 redirect or proxy header 설정을 통해 백엔드/프론트에서 param 전달
    - nginx ip 식별
      - $remote_addr & map을 통한 IP 변수화
      - GeoIP2 module(.mmdb)을 통한 접근 IP 변수화

<br />

---

> ### Options

    ```java
    location/ {
      proxy_pass http://localhost:8080;
      proxy_http_version  1.1;
      proxy_cache_bypass  $http_upgrade;

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

<br />

---

 ### default.conf & nginx.conf

  - #### nginx.conf
  
    - nginx.conf : nginx의 기본 설정을 담당
    - worker_processes: worker_process로 유입되는 이벤트를 처리하는 프로세스 수(보통 cpu 코어만큼 추천)
    - error_log: nginx의 에러 로그가 쌓이는 경로
    - pid: nginx의 프로세스 아이디 저장 경로
    - worker_connections: worker process가 동시에 처리할 수 있는 접속자 수: 기본은 1024
    - include: 포함시킬 외부 파일
    - default_type: 웹 서버의 기본 content type 정의
    - log_format: 로그 형식을 지정
    - access_log: 접속 로그가 쌓이는 경로
    - sendfile: sendfile() 함수의 사용 여부
    - keepalive_timeout: 클라이언트에서 연결이 유지될 시간을 정의함(기본: 65)
  
  - #### default.conf
  
    - default.conf : nginx.conf에서 include해서 사용된다. 서버 설정 관련 파일이며 default2.conf, default3.conf는 여러 개의 파일을 추가해서 서버 관련 설정을 추가로 포함시킬 수 있다.
    - listen : 해당 포트로 들어오는 요청을 해당 server {} 블록의 내용에 맞게 처리하겠다는 것을 뜻한다.
    - server_name : 호스트 이름을 지정한다. 가상 호스트가 있는 경우 해당 호스트명을 써넣으면 된다. 만약 로컬에서 작업하고 있는 내용을 nginx를 통해 띄우려고 하는 경우에는 localhost라고 적으면 된다.
    - error_page : 요청결과의 http 상태코드가 지정된 http 상태코드와 일치할 경우, 해당 url로 이동한다. 보통 403, 404, 502 등의 에러처리를 위해 사용한다.
    - location / : 처음 요청이 들어왔을 때 ( server_name이 127.0.0.1인 경우 -> 127.0.0.1로 요청이 들어왔을 때 ) 보여줄 페이지들이 속해있는 경로와 초기 페이지인 index를 지정해준다. / url로 접속했을 경우 index.html, index.htm로 정의된 파일을 보여준다.
    - ssl_certificate: 생성된 인증서 설정 경로
    - upstream: proxy_pass 지시자를 활용해 nginx가 받은 요청을 넘겨 줄 서버를 정의


  

<br />

---