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
  - 현재 프로젝트에서 Web Server로 nginx를 사용하고 있습니다. 현재 nginx.conf를 통해 nginx에서 제공해주는 다양한 기능들을 사용하고 있는데 특히 Reverse Proxy로써 Server 앞단에서 SSL처리, redirect, load balancing 등 많은 역할을 하고 있습니다. Nginx에 대해 더 자세히 알아보고 무중단 배포와도 연관하여 학습하고 정리해보고자 합니다.

- Web Server(Apache, Nginx)
  - Reverse Proxy(Client -> Web Server -> WAS)
    - 서버 부하 분산 : 들어오는 요청을 여러 대의 서버로 분산시켜 각 서버의 부하를 줄이고 서버의 가용성을 높여 안정적인 서비스를 제공
    - 보안 강화 : Reverse Proxy를 거쳐 서버로 전달되기 때문에 외부에서 직접 서버에 접근할 수 없다(SSL, Filtering...)
    - 캐싱 및 가속화 : 자주 사용되는 정적 파일(image, css, javascript..)을 캐시에 저장하여 빠르게 제공(응답 시간 단축, 서버의 부하 감소소)

<br />

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