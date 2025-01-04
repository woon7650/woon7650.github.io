---
title: "Remind of Aspect Oriented Programming"
excerpt: "[Java] Feature of AOP, Proxy Pattern"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2024-01-04
---


### 들어가면서

  - 지난 포스트에서 좋은 설계를 위한 OOP 특징, 설계 원칙(SOLID), Design Pattern에 이어서 이번 포스트에서 AOP에 대해서 한 번 더 상기해보고자 합니다.
  - Spring 3대 기술 중에 하나인 Proxy를 기반으로 동작하는 AOP에 대해 알아보고 동작 과정에 대해서 알아보겠습니다.

<br />

---

> ### AOP(Aspect Oriented Programming)
  - ### AOP?

    - Spring 3대 기술(DI, PSA, AOP)중 하나인 AOP는 Proxy 기반으로 동작한다.
    - 부가기능은 핵심 기능에 의존하여 작동한다.
    - 애플리케이션에 중복으로 존재하는 부가기능을 모듈화한 것을 Aspect라고한다.
    - Advice는 Aspect에 구현되어 있는 부가기능
    - Pointcut은 Aspect 모듈이 추가될 대상을 선정하는 로직을 담은 오브젝트
    - AOP : 핵심기능으로부터 부가기능들을 분리해 Aspect 모듈을 생성하는 프로그래밍 방식
  
> ### Design Pattern

  - #### Proxy Pattern
    - Client가 Target에 접근하는 방식을 변경하는 패턴
    - Target Object를 생성하지 않고 레퍼런스만 가지고 싶을 때 아주 유용
    - Target에 대한 Proxy를 가지고 있음으로써 Target에 대한 참조를 가진다.
    - ex. JPA lazy loading
      - Target의 참조가 일어나기전까지 DB SELECT가 발생하지 않는다

  - #### Decorator Pattern
    - Target의 부가 기능을 Proxy를 통해 추가하는 패턴
    - Target의 코드 수정은 필요 없음
    - 컴파일 시점에 Proxy와 Target의 관계가 정의되어 있지 않지만 Client가 동적으로 관계를 맺는다.
    - Proxy는 1개 이상 가능하며 이 때 각 Proxy를 하나의 Decorator라고 한다.
    - Decorator는 요청을 위임하는 대상이 Interface이기 때문에 대상이 Target인지 다른 Decorator인지 알지 못한다.

> ### Proxy

  - Client가 사용하려는 대상인 것처럼 행동해서 Client 요청을 받아 실제 객체에 요청을 위임하는 객체
  - Target은 클라이언트가 직접 사용하려고 했던 객체
  - Proxy는 주로 타깃의 기능을 확장(Proxy Pattern), 타깃에 대한 접근 방법을 제어(Decorator Pattern)하기 위해 사용한다.

> ### Proxy 종류


  - #### Dynamic Proxy
    - 클라이언트가 매번 Proxy(핵심 기능 구현체)를 작성해야하는 번거로움이 있음
      - Target 수만큼 Proxy class 수가 늘어남

    - JDK에서 제공하는 java.lang.reflect를 이용해 매번 Proxy를 작성하지 않고 런타임에 Proxy를 만들 수 있게 한다.
    - Proxy Factory가 동적으로 만드는 Object
    - Dynamic Proxy = Target Interface와 같은 타입
    - Proxy Factory에게 구현해야할 Interface 정보를 주면 Proxy Factory가 알아서 Proxy를 생성해준다
    - 단점 :
      - Target Object의 메소드 별로 서로 다른 부가기능을 제공하고 싶다면?
      - 하나의 Target에 여러 부가기능을 적용한다면?
    - 모두 Target Object 전체에 Proxy를 적용해서 사용
    
  - #### CGLIB

> ### Proxy Bean


  - #### Factoy Bean

  - #### Advisor(ProxyFactoryBean)
    - Dynamic Proxy는 Spring DI를 위해 Spring Factory Bean을 사용했다 -> ProxyFactoryBean을 이용하여 부가기능 로직과 Proxy를 제공하는 빈을 완벽하게 분리 가능하다.
    - Advisor = Advice + Pointcut
      - Advice : Target Object에 추가할 부가기능을 담은 Object
      - Pointcut : 부가기능을 적용할 Target읠 메소드를 선정하는 알고리즘을 담은 Object
      - Proxy는 Pointcut으로 Advice 적용 대상인지 확인 후 Advisor에게 부가기능을 요청


  - ### DefaultAdvisorAutoProxyCreator(빈 후처리기)
    - Advisor를 탐색해 자동으로 Proxy를 생성하는 자동 Proxy 생성기
    - Pointcut을 기반으로 Proxy 적용 대상인지 확인 -> Proxy 적용 대상이면 기존의 Bean을 Proxy로 교체하여 Conatiner에 반환

  - ### AspectJ
    - 대표적인 AOP 프레임워크 -> Target Object의 바이트 코드를 직접 조작한다.