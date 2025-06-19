---
title: "Docker installation and setting process through wsl"
excerpt: "[DevOps] About Docker installation and setting process through wsl"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2025-06-18
---

### 들어가면서

  - Situation
    - 새로운 토이프로젝트를 진행함에 있어서 Docker를 사용해서 진행을 해보고자 합니다. 
  - Condition
    - 상황에 따라 개인은 다른 PC에서 작업 가능(개인당 2대 이상의 컴퓨터에서 개발을 진행 가능성)
    - 2명 이상의 팀원들과 개발을 진행
    - 우선 local로 진행하기 때문에 EC2까지는 불필요 -> 공통으로 사용해야하는 DB만 RDS를 사용하기로 결정
  - Consideration
    - 나머지 개발 환경에 필요한 Tool 및 Library(Nginx, Kafka, Prometheus, Grafana)..은 PC마다 local download보다는 Docker를 이용하여 동일한 버전의 동일한 Tool을 사용하기로 결정
    
  - Conclusion
    - RDS(DB) + Git(Source) + docker-compose.yml을 통해서 서로 다른 모든 PC 같은 공유되는 DB 및 여러 가지 Tool을 이용하여 동일한 환경에서 공유하면서 개발을 가능하게 된다

<br />

---

- ### Docker
  - Docker : Container 기술을 사용하는 Platform/Tool
    - Image 생성/실행/관리
  - Image : 실행 가능한 코드와 환경/설정의 정적인 스냅샷
    - 생성 : `docker pull`, `docker build`
  - Container : Docker가 실행하는 가상화된 실행 환경 instance(Image를 실행한 동적인 Instance)
    - 생성 : `docker run`, `docker compose up`(docker-compose.yml이 위치한 경로에서)
    - Image만 pull 해둔 상태에서 Container는 존재하지 않음

- ### Process
  1. Docker Desktop 설정 및 사용(WLS or Hyper-V)
      - Docker Desktop(GUI + Docker Engine) : 내부에서 WSL2 기반으로 실행
      - WSL2 : Linux 배포한으로 Container 실행 환경
  2. Container에서 사용할 Image를 미리 pull
  3. docker-compose.yml 작성
  4. 사용할 image의 설정 파일(nginx.conf, prometheus.yml..)을 docker-compose.yml가 있는 경로에 미리 준비
  4. docker compose up-d 실행

<br />

---


- ### WSL2
  - Windows 내에서 Linux Kunnel을 실행할 수 있는 Microsoft Official Hypervisor
  - Docker는 Linux Kunnel 위에서 동작(Window에서는 WSL2 또는 Hyper-V 필요)
    - `Docker Desktop + WSL2`
  - Window 버전에 따라 WLS or Hyper-V 사용(해당 환경은 Window 10 Home -> WSL2 기반 Docker Desktop 사용)
  - PowerShell
    ```
    wsl --install
    ```  

- ### Docker Desktop
  - download : `https://www.docker.com/products/docker-desktop/`
  - "Use WLS 2 instead of Hyper-V" 활성화
  - Setting > Resources > WSL integration > Enable integration with my default WSL distro 활성화

  - #### Conception
    - Docker Engine : WSL2 위에서 돌아가는 Container Runtime
    - Docker CLI: PowerShell/WSL/CMD 어디서든 사용 가능(docker 명령어)
    - Docker Compose : 여러 Container를 묶어서 실행하는 도구
    - Dashboard : Image/Container 상태 시각화/log 보기

  - #### Check Installation
    - WLS2 installation check
    - 실행중 : `STATE : Running` / WLS 사용 : `VERSION : 2`
      ```bash
      wsl --list --verbose
      ```
    - Docker installation check
      ```bash
      docker info
      docker version
      ```

- ### Image pull

  - #### Container 실행에 필요한 이미지를 직접 download
    - image가 없을 경우에 `docker-compose up`를 진행하게 되면 느리게 download -> Image를 미리 pull할 것을 권장
    - `docker pull` : Docker Hub에서 Image를 download
    - bash
      ```
      # Redis
      docker pull redis:alpine

      # Nginx
      docker pull nginx:latest

      # Prometheus
      docker pull prom/prometheus

      # Grafana
      docker pull grafana/grafana

      # Kafka
      docker pull bitnami/kafka
      docker pull bitnami/zookeeper
      ```
    - result
      ![image info](/assets/img/docker_image.png)
      <img src="/assets/img/docker_image.png" alt="" width="0" height="0"> 

  - #### Container 생성 시 Image 일괄적으로 pull
    - `docker-compose.yml`을 기반으로 Container에 필요한 이미지들을 설정된 정보에 따라 일괄적으로 download


    - `docker-compose.yml` 작성
      ```yml
      version: '3.8'

      services:
        redis:
          image: redis:alpine
          container_name: redis
          ports:
            - "6379:6379"

        zookeeper:
          image: bitnami/zookeeper:latest
          container_name: zookeeper
          environment:
            - ALLOW_ANONYMOUS_LOGIN=yes
          ports:
            - "2181:2181"

        kafka:
          image: bitnami/kafka:latest
          container_name: kafka
          depends_on:
            - zookeeper
          ports:
            - "9092:9092"
          environment:
            KAFKA_CFG_ZOOKEEPER_CONNECT: zookeeper:2181
            KAFKA_CFG_LISTENERS: PLAINTEXT://:9092
            KAFKA_CFG_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
            ALLOW_PLAINTEXT_LISTENER: yes

        prometheus:
          image: prom/prometheus
          container_name: prometheus
          ports:
            - "9090:9090"
          volumes:
            - ./prometheus.yml:/etc/prometheus/prometheus.yml

        grafana:
          image: grafana/grafana
          container_name: grafana
          ports:
            - "3000:3000"
          volumes:
            - grafana_data:/var/lib/grafana

        nginx:
          image: nginx:latest
          container_name: nginx
          depends_on:
            - redis
          ports:
            - "8080:80"
          volumes:
            - ./nginx.conf:/etc/nginx/nginx.conf:ro

      volumes:
        grafana_data:

      ```

    - result
      ![image info](/assets/img/docker_image2.png)
      <img src="/assets/img/docker_image2.png" alt="" width="0" height="0"> 


- ### Container

  - #### Command
    - docker 실행/중지
      ```
      docker compose up -d
      docker compose down
      ```
    - docker 상태 확인(각 Container의 상태 (Up, Exited, Restarting) 확인)
      ```
      docker compose ps
      docker ps
  - #### Process of `docker compose up -d` 

    1. docker-compose.yml 파싱
        - services, image, ports, volumes, environment, depends_on.. 메모리에 구성
    2. network 생성
        - Compose는 각 프로젝트에 대해 고유한 Network(Bridge 타입) 생성
        - Container간 `ContainerName:port` 형태로 통신 가능하게 함(kafka -> zookeeper:2181)
        - bash
          ```
          docker network create [DirectoryName]_default
          ```
    3. Docker Volume 생성
    4. Image Pull / Check Local Cache
        - Local Image 존재 : 재사용
        - Local Image 존재하지 않음 : DockerHub/Bitnami.. 에서 자동 pull 
    5. Container 생성 순서 지정(`depends_on`)
        - 의존 대상 Container부터 생성(kafka : zookeeper 이후에 생성)
    6. Container Create + Start
    7. Port Binding
      - 호스트(Windows/WSL2)의 port를 Container port에 bind
        - 이미 사용 중일 경우 : `Ports are not available: bind: An attempt was made to access a socket...` Error 발생
    8. Mount 경로 검증
        - docker-compose.yml에 명시된 경로의 존재 여부 확인
        - 위의 docker-compose.yml 기준 docker > nginx.conf, prometheus.yml
    9. 백그라운드 모드(-d)로 실행
        - 모든 Container는 Daemon mode로 background에서 실행
    10. 정상 실행 상태 확인

        ![image info](/assets/img/docker_run.png)
        <img src="/assets/img/docker_run.png" alt="" width="0" height="0"> 

        ![image info](/assets/img/docker_run2.png)
        <img src="/assets/img/docker_run2.png" alt="" width="0" height="0"> 

<br />

---

- ### Docker <-> Window 네트워크 접근
  - Window -> Docker : `localhost:<지정된 port>` 사용
  - Docker -> Window : `host.docker.internal` 사용
    -	WSL2에서 Windows Host에 접근할 때 사용하는 내부 DNS 주소

- ### Windows에서 Docker Image에 접근하는 Port

  - Nginx -> localhost:8080
  - Prometheus -> localhost:9090
  - Grafana -> localhost:3000
  - Redis -> localhost:6379
  - Kafka -> localhost:9092

<br />

---