---
title: "Load Library Exception : No package_jni in java.library.path"
excerpt: "[Java] JNI DLL Exception occured during DRM link"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2025-05-19
---


### 들어가면서
  - 지난 번 포스트에서 알아본 DRM 관련 연동을 하면서 발생한 자그마한 Exception과 해결 방법에 대해 다뤄 보고자 합니다.
  - 또한 **DLL, JNI, JAR**의 개념에 대해서도 정리를 해보고자 합니다.

<br />

---

- ### ⚠️ Situation
  - 현재 시스템에서 기본 DRM SoftCamp를 사용하고 있으며 최근 특정 서비스에 Fasoo DRM 연동을 추가로 진행하면서 발생한 이슈입니다
    ```java
    Exception : load library Exception : no pkg_jni in java.library.path
    com.fasoo.fcwpkg.packager.FassoPackagerJNI.N_DestroyObject(J)V
    ```
  - ⚠️ 핵심 : **System.loadLibrary("pkg_jni")**를 호출 -> **java.library.path**에서 **pkg_jni.dll**을 찾을 수 없음 -> **UnsatisfiedLinkError** 발생



- ### 💡 Solution

  - #### VM option에 java.library.path를 설정

    - Eclipse 기준 -> Run > Run Configuration > Arguments > VM Arguments
      > -Djava.library.path="해당 library가 존재하는 directory"
    

    - Tomcat(Window) 기준 : JVM 실행 옵션을 직접 설정 -> /bin/setenv.bat
      > set "JAVA_OPTS=%JAVA_OPTS% -Djava.library.path=해당 library가 존재하는 directory"

  - 해당 경로에 **pkg_jni.dll**이 존재하게 하면 해결완료


- ### 🔁 Process

  - Application 실행 -> JVM(Java Virtual Machine) 기동
  - JVM은 **classpath** 설정을 바탕으로 필요한 **.class** 또는 **.jar** 파일을 ClassLoader를 통해 로딩(jar안에 포함된 Java class들도 로딩)

  - 로딩된 Java class 내부에서 JNI(Java Native Interface)를 통해 C/C++로 구현된 native 메소드를 호출
    ```java
    System.loadLibrary("pkg_jni");
    ```
    - OS에 **pkg_jni.dll**을 로딩하라고 요청
    - JVM은 내부적으로 **java.library.path**에 설정된 Directory를 통해 **pkg_jni.dll**를 찾습니다
    - **java.library.path**가 설정되어 있지 않으면 운영체제의 **PATH** 환경 변수에서 탐색
  - **pkg_jni.dll**을 발견하면 JVM은 해당 DLL을 메모리에 로딩
    - 해당 dll이 의존하고 있는 다른 dll이 있을 경우 그 의존 dll들까지 함께 찾고 로딩
    - 만약 의존 dll을 찾지 못하면 **UnsatisfiedLinkError : Can't find dependent libraries** Exception 발생
  - **pkg_jni.dll**이 성공적으로 로딩되면 JVM은 해당 dll 내부의 C/C++ 구현 함수들을 Java 메서드에 바인딩
  - Java 코드에서 native method가 호출되면 JVM은 **JNI Bridge**를 통해 C/C++ method를 호출
    - Java -> C코드로 제어권이 넘어가며 메모르/리소스를 공유하고 처리 결과를 Java로 반환 


<br />

---


- ### ✅Conclusion
  - 개발도 중요하겠지만 어플리케이션이 돌아가는 과정과 관련된 기본적인 개념에 대해서 한 번씩 정리하는 과정을 통해서 다음에 비슷한 jar와 dll 관련된 이슈가 생기더라도 빠르게 대처할 수 있을 것 같습니다.  