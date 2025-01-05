---
title: "Spring AOP and AspectJ"
excerpt: "[Spring] AsepctJ and Limit of Spring AOP with Self Invocation"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2024-01-05
---

##### RELATED POST
- [Spring AOP](blog/Spring-AOP/)
- [Spring Proxy](blog/Spring-PROXY_CGLIB/)

<br />

---

### 들어가면서
  - 지난 포스트에서 좋은 설계를 위한 OOP 특징, 설계 원칙(SOLID), Design Pattern에 이어서 이번 포스트에서 AOP에 대해서 한 번 더 상기해보고자 합니다.

<br />

---

> ### AOP(Aspect Oriented Programming)

  - ### AOP??
    > 관점 지향 프로그래밍 : 흩어진 관점(Aspect)를 모듈화하여 핵심 비즈니스 로직을 해치지 않고 재사용하는 프로그래밍 기법
    - 애플리케이션에는 공통적으로 존재하는 코드(트랜잭션 처리, 로깅, 보안처리, 인증/인가)들이 존재한다.
    - 부가 기능은 핵심 기능에 적용되어야 하기 때문에 핵심 기능에 의존하여 동작한다.
    - 즉, 핵심 로직/기능으로부터 부가기능들을 분리해서 Aspect 모듈을 만들어 프로그래밍 하는 방법이다.

  - ### Related Terms
    - Aspect : Target에 적용될 부가 공통 관심 사항(Advice + Pointcut)
    - Target : Aspect(공통관심사)를 적용할 핵심로직을 가진 객체
    - Advice : JointPoint에 적용될 부가 로직
    - JointPoint : Advice를 적용할 수 있는 지점(메서드, 필드, 객체, 생성자)
    - Pointcut : JointPoint의 정규식으로 Advice를 적용할 JointPoint을 선별하는 표현식
    - Weaving : Pointcut으로 결정된 Target에 부가기능(Advice)을 부여하는 과정

  - ### Implement of AOP
    - Spring AOP
      - Spring IoC에서 간단한 AOP 구현을 제공하는 것을 목표
      - Spring Container가 관리하는 Bean에만 적용이 가능
    - AspectJ
      - 완전한 AOP 솔루션을 제공하는 것을 목표
      - 모든 도메인 객체에 적용이 가능

> ### Proxy

  - Client가 사용하려는 대상인 것처럼 행동해서 Client 요청을 받아 실제 객체에 요청을 위임하는 객체
  - Target은 Client가 직접 사용하려고 했던 객체이며 Proxy는 Target의 대리자 역할이다.
  - Proxy는 주로 타깃의 기능을 확장(Proxy Pattern), 타깃에 대한 접근 방법을 제어(Decorator Pattern)하기 위해 사용한다.

  - #### Dynamic Proxy
    - Java에서 제공하는 java.lang.reflect.Proxy를 사용하여 Interface를 기반으로 Proxy를 생성한다.
      - Interface에 적혀있는 모든 Method에 대하여 Proxy를 적용시킨다.
    - InvocationHandler의 invoke()를 @Override하여 실제 객체의 정보를 받아오고 조작한다.
    - Proxy를 하나하나 직접 만드는 것이 아닌 Runtime 시점에 Proxy를 생성할 수 있다.
    
  - #### CGLIB Proxy
    - Java Byte Code를 조작하여 Proxy를 생성해주는 코드 생성 라이브러리이다.
      - 특정 Method에 대하여만 추가 동작을 하도록 만들 수 있다.
    - Target 객체를 직접 상속받아서 Proxy를 생성한다.

  - ### Difference

    - Dynamic Proxy : Interface 기반으로 Target이 하나 이상의 Interface를 구현
    - CGLIB Proxy : Interface를 구현하지 않음
      - ###### Dynamic Proxy, CGLIB Proxy에 대한 자세한 내용은 위에 포스트 링크를 참고

    ![image info](/assets/img/AOPProcess.png)
    <img src="/assets/img/AOPProcess.png" alt="" width="0" height="0">


> ### Weaving

  - #### Category of Weaving

    - Runtime Weaving(RTW)
    - Compile Time Weaving(CTW) : Target 코드가 JVM상에 올라갈 때(컴파일 시점) Byte Code를 직접 조작하여 Target의 Method내에 Advice 코드를 삽입시켜준다.
      - AspectJ Compiler를 이용
      - **컴파일 시점 Weaving**
    - Post Compile Weaving(PCW) : 컴파일된 바이너리 코드 또는 JAR에 포함된 소스에 위빙한다.
      - JAR를 이용
      - **컴파일 직후 Weaving**
    - Load Time Weaving(LTW) : 바이트 코드에 직접적으로 조작하지 않고 오브젝트가 메모리에 올라가는 과정에서 위빙힌다.
      - Class Loader를 이용

  - #### Difference of Weaving

    - Compile Time : CTW < PCW < LTW
    - Runtime : LTW < PCW < CTW


> ### Spring AOP

  - Proxy 기반으로 Runtime Weaving 방식을 따르며 IoC Container에서 Bean을 생성하는 시점에 AOP를 적용할지 여부를 판단하여 Proxy Bean을 생성한다.
  - Interface를 구현한 경우 JDK Dynamic Proxy, Interface를 구현하지 않은 경우 CGLIB Proxy로 Proxy Bean을 생성한다.
  - 지정된 Method가 호출될 때 해당 Method를 가로채서 부가 기능들을 추가할 수 있도록 지원한다.
  - Example
    - @Transaction, @Cacheable, @Async..


  - #### RunTime Weaving

  - #### Self-Invocation
    - Proxy 메커니즘을 기반으로 한 AOP 기술에서 발생할 수 있는 issue
    - JDK Dynamic Proxy, CGLIB Proxy 모두에서 Self-Invocation이 발생한다.
      - CGLIB도 JDK Dynamic Proxy를 기반으로 설계된 구조를 통해 동작

  - #### Solution of Self Invoction
    - AOP Context
    - IoC Container Bean 활용
    - AspectJ Weaving

> ### AspectJ

  - #### Weaving

    - Compile Time Weaving(CTW)
    - Post Compile Weaving(PCW)
    - Load Time Weaving(LTW)





<br />

---

### Reference

- https://kghworks.tistory.com/161
- https://kghworks.tistory.com/162
- https://gmoon92.github.io/spring/aop/2019/05/24/aspectj-of-spring.html
- https://gmoon92.github.io/spring/aop/2019/04/01/spring-aop-mechanism-with-self-invocation.html
- https://minhye0k.github.io/spring-aop-%EC%99%80-self-invocation-%EC%9D%80-%EB%AC%B4%EC%8A%A8-%EA%B4%80%EA%B3%84%EA%B0%80-%EC%9E%88%EB%8A%94%EA%B1%B8%EA%B9%8C
- https://gmoon92.github.io/spring/aop/2019/05/24/aspectj-of-spring.html
- https://velog.io/@youjung/Spring-AOP