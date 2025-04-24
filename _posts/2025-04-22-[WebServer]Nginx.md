---
title: " Reverse Proxy Nginx Conception, Configuration"
excerpt: "[Web Server] About Reverse Proxy Nginx Conception, Configuration"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2025-04-21
---


### 들어가면서
  - 현재 프로젝트에서 Web Server로 nginx를 사용하고 있습니다. 현재 nginx.conf를 통해 nginx에서 제공해주는 다양한 기능들을 사용하고 있는데 특히 Reverse Proxy로써 Server 앞단에서 SSL처리, redirect, load balancing 등 많은 역할을 하고 있습니다. Nginx에 대해 더 자세히 알아보고 무중단 배포와도 연관하여 학습하고 정리해보고자 합니다.

- Web Server(Apache, Nginx)
  - Reverse Proxy(Client -> Web Server -> WAS)
    - 서버 부하 분산 : 들어오는 요청을 여러 대의 서버로 분산시켜 각 서버의 부하를 줄이고 서버의 가용성을 높여 안정적인 서비스를 제공
    - 보안 강화 : Reverse Proxy를 거쳐 서버로 전달되기 때문에 외부에서 직접 서버에 접근할 수 없다(SSL, Filtering...)
    - 캐싱 및 가속화 : 자주 사용되는 정적 파일(image, css, javascript..)을 캐시에 저장하여 빠르게 제공(응답 시간 단축, 서버의 부하 감소소)

<br />

---
> ### Load Balancing

  - Client 요청이 많아져서 Server에 부하가 많아질 경우 Request들을 여러 Server에 균형있게 분산시켜 각 Server가 원활히 동작할 수 있도록 한다
  - #### Scale Up
    - 서버 자체의 성능을 향상
    - 하드웨어의 성능을 무한정 올리기에는 한계가 있으며 시스템 장애에 취약하다(SPOF, Single Point Of Failure)
  - #### Scale Out
    - 비슷한 서버를 증설하여 Request를 분배
    - 다수의 사용자 요청을 처리할 수 있으며 무중단 서비스를 제공할 수 있지만 세션 불일치, 서버 관리 등 기술 난이도가 존재

  - #### Load Balancer
    ![image info](/assets/img/loadBalancing.png)
    <img src="/assets/img/loadBalancing.png" alt="" width="0" height="0">    
    - Scale Out과 Load Balancer를 이용하여 성능 및 가용성 이점과 Reverse Proxy를 통해 보안을 향상할 수 있다다

  - #### SSL OffLoading
    - 암호화 및 복호화 작업을 각 Server가 아닌 Load Balancer에서 처리한다

    - SSL OffLoading을 적용하면 Load Balancer에 SSL 인증서가 존재하기에 SSL HandShke 과정이 효율적으로 진행된다
    - 암호화된 요청이 아닌 평문으로 Web Server에 전달 하기 때문에 네트워크 대역폭을 더 효율적으로 사용할 수 있다
    - Scale out시에 별도의 SSL 인증서를 추가하지 않아도 되며암호화, 복호화에 CPU를 사용하지 않아도 된다
      
<br />

---
> ### Load Balancing Algorithm
 
 - 비즈니스 성격에 맞게 Load Balancing Algorithm의 각각의 장단점을 파악하고 구축해야 한다

  - #### Round Robin(라운드 로빈)
    - 다수의 서버에 **순서대로** 요청을 할당
    - 서버에 균등하게 요청을 분배할 수 있지만 각 서버의 처리량, 서버의 상태, 작업 부하 등을 고려하지 못한다

    ```java
    //default : round-robin
    upstream myserver {
        server localhost:8080;
        server localhost:8081;
        server localhost:8082;
    }
    ```

  - #### Least Connection(최소 연결결)
    - **Connection이 가정 적은 서버**에 요청을 전달
    - 각 서버의 Connection 수를 계속해서 추적해야 하는 복잡성이 존재하고 서버 응답 시간, 상태태를 고려하지 못한다

    ```java
    upstream myserver {
    	least_conn;
        server localhost:8080;
        server localhost:8081;
        server localhost:8082;
    }
    ```


  - #### Weight Ratio(가중치 비율)
    - 각 서버의 처리 능력을 고려하여 서버가 가질 수 있는 **처리량, Connection 비율 가중치**를 토대로 부하를 분산
    - 각 서버의 성능에 따라 효율적인 처리가 가능하지만 최적 서버 가중치를 유지하는데 어려움이 있다

    ```java
    upstream myserver {
        server localhost:8080 weight=3;
        server localhost:8081 weight=2;
        server localhost:8082 weight=1;
    }
    ```

  - #### IP Hash(IP 해시)
    - Round Robin, Least Connection, Weight Ratio는 각 서버마다 세션 불일치 문제가 발생할 수 있다(세션 정보가 저장되있는 서버가 아닌 다른 서버에도 같은 세션 정보가 저장될 때 중복 데이터 문제가 발생생)
    - **패킷의 IP주소를 해싱하여 결과 해시 값**에 해당하는 서버가 처리하는 것을 보장할 수 있다
    - 적은 수의 Client와 많은 요청을 처리하는 경우 특정 서버에 부하가 몰릴 가능성이 있다

    
    ```java
    upstream myserver {
	    ip_hash;
        server localhost:8080;
        server localhost:8081;
        server localhost:8082;
    }
    ```

  - #### Least Response Time
    - **응답 시간이 가장 빠른 서버**에 우선적으로 요청을 할당
    - 서버 응답 시간 파악을 위한 모니터링 시스템이 필요하며 서버 상태, 처리량을 고려하지 못한다


<br />

---

> ### Nginx Conf

- #### Main Context
    - 전역 설정이 정의되는 최상위(top-level) 컨텍스트

    - user : Nginx worker process에 대한 사용자 및 그룹을 정의
    - worker processes : work process 지정 
    - pid : PID 파일의 위치 설정

- #### Events Context
    - 연결 처리와 관련된 설정을 정의

    - worker_connections : 작업자 당(per worker) 최대 연결 수를 설정
    - user : "epoll"과 같은 사용할 이벤트 기반 모델 지정

- #### HTTP Context
    - 웹 서버 구성에 사용됩

    - include : 추가 구성 파일 포함
    - sendfile : sendfile() 사용을 활성화 또는 비활성화화

- #### Server Context
    - HTTP Context 내에서 서버 수준(server-leve) 설정을 지정

    - listen : IP Address와 Port 지정
    - server_name : 도메인 이름을 정의

- #### Location Context
    -  URL 패턴을 기반으로 요청이 처리되는 방식을 구성

    - proxy_pass : 요청을 다른 서버로 전달
    - root : 요청의 Root Directory를 설정

    
- #### Stream Context
    -  TCP/UDP 프록시 구성에 사용용

    - stream : Stream 관련 구성을 포함
    - server : Stream 처리를 위한 server block 정의


> ### default.conf & nginx.conf

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