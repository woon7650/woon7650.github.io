---
title: "Dynamic Proxy & CGLIB of Spring AOP"
excerpt: "[Spring] Spring AOP Dynamic Proxy 와 CGLIB"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2024-09-29
---


#### 0. 들어가면서

  - 이직 준비를 하면서 면접에서 자주 등장하는 Backend CS 관련 지식을 정리하고 자세히 알아보고자 합니다.
  - Spring AOP와 Proxy 구현에 대해서 알아보고자 합니다.



> ### AOP & Proxy

  > What is Aspect-Oriented Programming (AOP)? 
  Aspect-Oriented Programming (AOP) is a programming paradigm that enables the modularization of cross-cutting concerns in software applications. Cross-cutting concerns are aspects of your application that affect multiple parts of the codebase. These can include logging, security, transactions, and error handling. AOP allows you to separate these concerns from the core application logic, making your code more maintainable and less cluttered.

  > AOP란?
  소프트웨어 애플리케이션에서 크로스 커팅 문제를 모듈화할 수 있는 프로그래밍 패러다임입니다. 크로스 커팅 문제는 코드베이스의 여러 부분에 영향을 미치는 애플리케이션의 측면입니다. 여기에는 로그, 보안, 트랜잭션, 오류 처리 등이 포함될 수 있습니다. AOP를 사용하면 이러한 문제를 핵심 애플리케이션 로직과 분리하여 코드를 더 유지 관리하기 쉽고 덜 어수선하게 만들 수 있습니다.

  ![image info](/assets/img/AOP.png)
  <img src="/assets/img/AOP.png" alt="" width="0" height="0">


  - Spring에서 Proxy를 바탕으로 관심사를 추출하는 AOP를 제공한다.(Runtime Weaving 방식, Proxy 패턴)
  - Runtime Weaving : 동적으로 생성된 Proxy Bean은 Target의 메소드가 호출되는 시점에 부가기능을 추가할 메소드를 자체적으로 판단하고 가로채어 부가기능을 주입한다.
  - Proxy : 실제 Target의 기능을 대신 수행하면서 기능을 확장하고 추가하는 실제 객체
    - JDK Dynamic Proxy : **Reflection**을 이용해 Proxy 객체를 생성
    - CGLIB Proxy : **Enhancer** Library를 통해 Byte Code를 조작하여 생성
  - 위의 두 가지 Proxy 방식을 통해 Proxy Bean을 생성한다.

<br />

---

> ### JDK Dynamic Proxy

  - Interface 기반으로 Proxy를 생성해주는 방식
  - Invocation Handler를 상속받아 구현하며 Reflection을 사용하여 동적으로 Proxy를 생성한다.
    - Reflection : 구체적인 클래스 타입을 알지 못해도 클래스의 정보에 접근할 수 있게 해주는 JAVA API
  - **Java.lang.reflect.Proxy** 클래스의 **newProxyInstance()**를 이용해 Proxy 객체를 생성한다.

  - #### Process

    - Target의 Interface를 자체적인 검증 로직은 통해 ProxyFactory에 의해 Target의 Interface를 상속한 Proxy 객체 생성
    - Proxy 객체에 InvocationHandler를 포함시켜 하나의 객채로 반환




<br />

---


> ### CGLIB Proxy(Code Generator Library)

  - Enhancer를 바탕으로 Proxy를 구현하는 방식
  - Extends(상속)을 이용해 Proxy화 할 메서드를 @Override하는 방식
    - **final**, **private**을 사용할 경우 Proxy에서 해당 메소드에 대한 Aspect를 적용할 수 없다.
  - **ASM**을 사용하여 Byte Code를 조작해 Proxy 생성
    - ASM : Java Byte Code 조작 및 분석 프레임워크
  - MethodInterceptor/InvocationHandler 두 가지 방식으로 구현할 수 있다.

  - #### Process

    - **net.sf.cglib.proxy.Enhancer** 클래스를 사용하여 원하는 Proxy 객체 생성
    - **net.sf.cglib.proxy.Callback**을 사용하여 Proxy 객체 조작



<br />

---


> ### Dynamic Proxy vs CGLIB Proxy

  ![image info](/assets/img/AOPproxy.png)
  <img src="/assets/img/AOPproxy.png" alt="" width="0" height="0">


  - JDK Dynamic Proxy
    - Interface 대상 객체만을 Proxy로 생성할 수 있다.
    - Spring(Default)
    - Spring AOP(JoinPont) : InvocationHandler
    - Spring AOP(PointCut) : MethodMatcher
    - Spring AOP(Advice) : invoke Method

  - CGLIB Proxy
    - Class 대상 객체를 Proxy로 생성할 수 있다.
    - Spring Boot 2.0 이후(Default)
    - Spring AOP(JoinPont) : MethodInterceptor
    - Spring AOP(PointCut) : MethodMatcher
    - Spring AOP(Advice) : Interceptor Method



<br />

---

### Reference


- https://velog.io/@suhongkim98/JDK-Dynamic-Proxy%EC%99%80-CGLib
- https://huisam.tistory.com/entry/springAOP
- https://steady-coding.tistory.com/608
- https://gmoon92.github.io/spring/aop/2019/04/20/jdk-dynamic-proxy-and-cglib.html
- https://ones1kk.tistory.com/entry/Spring-Spring-AOP2
- https://yeonyeon.tistory.com/289
- https://velog.io/@gmtmoney2357/%EC%8A%A4%ED%94%84%EB%A7%81-%EB%B6%80%ED%8A%B8-%EB%8F%99%EC%A0%81-%ED%94%84%EB%A1%9D%EC%8B%9C-%EA%B8%B0%EC%88%A0CGLIB-ProxyFactory