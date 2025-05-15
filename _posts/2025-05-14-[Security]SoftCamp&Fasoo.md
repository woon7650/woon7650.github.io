---
title: "Consideration of Using Multi DRM Library in Service"
excerpt: "[Security] Consideration of Using Softcamp, Fasoo in Service"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2025-05-14
---


### 들어가면서
  - ✅ Issue
    -  DRM(Digital Rights Management) : 디지털 콘텐츠의 저작권 관리를 위한 기술로 대표적인 솔루션으로 SoftCamp, Fasoo, MarkAny가 있습니다
    - 현재 서비스에서는 SoftCamp DRM을 사용 중입니다. 특정 지사에서는 Fasoo DRM으로 문서를 관리합니다. 해당 지사에서 서비스를 이용하려고 할 때 항상 서비스에서는 SoftCamp DRM이 암호화 되어있는 상태로 문서가 존재해야 합니다
    - 서비스 내에 파일을 업로드 : Fasoo 복호화 -> SoftCamp 암호화 -> 서버에 저장
    - 서비스 내에 파일을 다운로드 : SoftCamp 복호화 -> Fasoo 암호화 -> Client에 저장

    - 보통 한 가지의 DRM을 사용하지만 두 가지 DRM을 사용하는 Reference나 사례가 존재하지 않음
  - ⚠️ Consideration 
    - 한 개의 WAS에서의 두 개의 DRM을 통한 암/복호화
    - 두 개의 WAS로 분리 및 API 통신을 통한 DRM 암/복호화

<br />

---

- ### Fasoo, SoftCamp의 공존의 문제점

  - #### ❌ Critical Problem
    - Fasoo 복호화 파일에 SoftCamp DRM 암호화 적용 → 포맷 이슈
      - Fasoo 복호화 시 일반 파일로 돌아오긴 하지만 일시적인 메타데이터/속성이 남아있을 수 있어 SoftCamp 암호화할 때 충돌이나 파일 파괴 가능성 존재
    - SoftCamp 복호화 파일에 Fasoo DRM 암호화 적용 → 중복 보호, 정책 꼬임
      - SoftCamp는 파일을 **자동 열람 제어, 파일 추적** 기능 포함 
      - 복호화 후 Fasoo 암호화를 진행하면 Fasoo는 SoftCamp 보호 흔적을 이해하지 못하므로 정책 충돌, 보안 정책 무력화 가능성 있음

  - #### ⚠️ Potential Problem

    - #### DRM은 메모리와 시스템 자원을 깊게 건드리는 보안 모듈
      - Fasoo : Java + 인증서 + Native 기반
      - SoftCamp : 시스템 서비스 + KeyManager + 라이브러리 기반
      - JVM이나 OS 수준의 자원에 접근하므로  자원 충돌 위험성 존재

    - #### 정책 충돌
      - SoftCamp, Fasoo는 모두 자체 정책 서버를 운영
      - 한 파일에 대해 SoftCamp는 열람 허용, Fasoo 차단 설정을 하면 특정 파일에 대한 정책 우선순위의 모순 발생

    - #### 성능 저하
      - SoftCamp, Fasoo DRM이 동시에 파일/메모리를 검사하면 서버 I/O가 느려지며 장애 발생의 원인 파악이 어려움

    
<br />

---


- ### 같은 WAS instance의 Port 분리

  - 위의 Fasoo, SoftCamp 공존 문제점과 동일(Port만 분리해서 해결되는 부분이 없음)

  - #### ❌ Problem
    - Tomcat 같은 WAS에서 Port를 다르게 설정하는 것은 단지 Server Socket 요청을 어디서 받을지를 설정한 것이다
    - 다른 Port를 사용하더라도 결국 하나의 JVM안에서 돌아가는 동일한 WAS instance
    - JVM의 ClassLoader, Memory, Native Library를 공유

  - #### ⚠️ Potential Problem
    - ClassLoader 충돌
      - SoftCamp, Fasoo 모두 native library 및 java class(jar, ll, so)를 사용 -> JVM 로딩에서 충돌
    - Native Library는 JVM 내 중복 로딩 불가 -> 하나의 WAS에서 두 DRM이 동시에 로딩하려는 경우 충돌
    - 인증서 경합
      - SoftCamp : System 기반
      - Fasoo : Java 인증서 기반
      - 둘 다 파일 기반 인증서 경로를 참조(속성 충돌, 경로 충돌 발생 가능)
    - Native Library Load 중복
      - JVM은 동일한 .dll, .so를 두 번 load하려고 하면 UnsatisfiedLinkError, LinkageError 등의 치명적인 오류 발생 가능
    - 리소스 경합
      - 암복호화 시 CPU/메모리 사용률이 높은 작업 -> 동일 JVM에서 처리할 경우 성능 하락
    - 장애 원인 추적이 불가능
      - 오류 발생 시 Fasoo, SoftCamp 어느 쪽에서의 장애인지 분석 불가능
      - 보안 로그 파일이 섞일 가능성 및 복구 불가능능

  - 가능하면 DRM 전용 서버를 분리하는 것이 좋음
  
  - #### ✅ Solution(서로 다른 WAS로의 분리)
    - 자원 충돌 방지 : 각 DRM은 다른 JVM에서 돌아가므로 리소스, 인증서, 키 충돌 없음
    - 서비스 안정성 확보 : 한 쪽 DRM만 장애 발생해도 전체 서비스 중단 없음, 한쪽 DRM만 업그레이드하거나 패치 가능
    - 보안 강화	: 각 DRM에 맞는 권한, 네트워크 정책 분리 가능
    - 장애 분석 간결 : 로그, 예외, 오류를 DRM별로 명확히 구분 가능

    
<br />

---

- ### 다른 WAS instance로의 분리

  - 동일 JVM에 두 개의 DRM이 공존할 때의 문제점들은 다 해결 가능

  - #### Architecture
    - WAS Instance A(SoftCamp 전용) – JVM 내에는 SoftCamp 라이브러리와 설정만 존재
    - WAS Instance B(Fasoo 전용) -	JVM 내에는 Fasoo jar 및 인증서만 존재
    - 🔁 서로는 API 호출로 연계	평문 파일을 임시 저장하거나 REST API 연동(RestTemplate, WebClient)

  - #### Flowchart

    - Upload
      Client (.fasoo) → A서버
      → A서버가 B서버로 복호화 요청 (Fasoo 해제)
      → A서버에서 SoftCamp 암호화
      → 저장 (.scenc)

    - Download
      Client 요청 → A서버
      → SoftCamp 복호화
      → B서버로 암호화 요청 (Fasoo 적용)
      → Client 전달 (.fasoo)


  - #### Consideration
    - 파일 연계 처리 : 중간 평문 처리 위치(Fasoo → SoftCamp 암복호화 사이의 파일은 임시 저장소를 사용해야 함)
    - 임시 파일 보안 : 평문 파일 저장소	중간 평문 파일은 디스크나 메모리상에 암호화된 상태로 유지 필요(보안 위험 발생 가능)
    - 처리 순서 보장 : 암/복호화 처리 순서 로직 상 Fasoo → SoftCamp 혹은 반대로 정확히 순서 관리해야 문제 없음


  - #### ⚠️ 위험성과 불확실성
    - 중간 평문 파일 유출 가능성 증가
      - Fasoo 복호화 ↔ SoftCamp 암호화 사이 구간에 파일은 반드시 한 번 평문 상태로 존재한다
      - 파일 유출 시 원문 노출	네트워크 전송 중 가로채거나 임시 저장소가 유출되면 평문 유출 가능
      - 기존에 DRM을 하나만 사용했을 경우우 암호화 파일만 처리 → 평문이 외부 노출될 일이 없었음
    
      - 💡 DRM 시스템은 "평문 노출 없이 처리"가 핵심인데 Fasoo와 SoftCamp를 병행할 경우 반드시 평문 상태로 처리하는 구간이 생기며 이는 DRM 본질과 충돌
    
    - 복잡한 시스템 아키텍처 → 장애 가능성 증가

      - 두 개의 WAS, DRM Library →	시스템 복잡도가 증가
      - 암/복호화 작업에 서버 간의 API 통신이 존재해서 통신 실패, 타임아웃, 인증 오류 등 네트워크 이슈로 인한 DRM 처리 실패 가능성 존재
      - 트랜잭션 처리 없음 : 암호화 1단계 성공 후 2단계 실패하면 불완전한 파일이 저장되거나 유출될 가능성

      - 💡시스템이 복잡해질수록 트러블슈팅 시간 증가 + 장애 원인 추적 어려움 → 운영 위험 증가
    
    - DRM 간 헤더 충돌 및 포맷 비호환 이슈
      - Fasoo → SoftCamp 순서	DRM 헤더 구조가 다르므로 파일 구조가 꼬일 가능성 있음
      - DRM 중복 적용 방지 필요	: 이미 DRM이 적용된 파일에 다시 DRM 적용 시 복호화 불가능한 상태 발생 가능
      - 💡 SoftCamp와 Fasoo는 각각의 파일 포맷 정의가 다름 → 만약 잘못된 순서로 적용 시 복구 불가능한 파일이 생성될 수 있음
    
    - DRM의 버전 관리 및 업데이트트
      - 업데이트 불일치 :	Fasoo와 SoftCamp의 버전 업그레이드 타이밍이 다르면 예상치 못한 충돌 발생 가능성 존재
      - 💡 복수 DRM을 사용할 경우 가장 큰 리스크는 한쪽 라이브러리 업데이트 시 다른 쪽이 깨지는 경우
    
    - 속도 저하 및 오류 처리
      - 업로드/다운로드 시 중간 API 통신 추가 및 절차 추가로 기존 방식보다 속도 저하 발생
      - 💡 사용자 입장에서는 문서 업로드/다운로드가 기존보다 처리 속도가 느리게 느껴질 수 있으며 파일 충돌 발생 시 파일 자체가 깨진 것처럼 느껴진다


    
<br />

---



- ### ✅Conclusion
  - 복호화 후 재암호화는 이론상으로는 가능하지만 해당 방식은 시스템템의 정상 동작을 보장할 수 없고 파일에 대한 정상적인 관리를 보장할 수 없다
  - 운영 가능한 구조이지만 비즈니스상 두 개의 DRM을 꼭 써야하는 상황이 아니라면 하나의 DRM으로 사용하는 것을 지향한다
  - 장기적으로 봤을 때 라이브러리 버전, 서버 간 API 통신, 속도 문제, 평문 파일이 존재할 때의 보안, 파일의 신뢰성을 생각했을 때 리스크가 존재하는 아키텍처이다