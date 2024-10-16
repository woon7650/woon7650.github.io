---
title: "Summary of Spring AOP"
excerpt: "[Spring] Summary of Spring AOP"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2024-10-16
---


#### 0. 들어가면서

  - Spring AOP에 대해서 간단하게 정리해보고자 합니다.


> ### AOP

  > 관점 지향 프로그래밍으로 **핵심기능과 부가 기능**으로 나누고 그 Aspect(관점)을 기준으로 각각 모듈화하는 방식을 뜻한다. Spring AOP는 기본적으로 Proxy 방식으로 동작한다. P

  - Proxy Pattern : 객체를 직접적으로 참조하는 것이 아니라 해당 객체를 대행(proxy)하는 객체를 통해 대상 객체에 접근하는 방식이다.
  - SRP(Single Responsibility Principle)에 따라 하나의 책임만을 갖게 설계된다.
  - Cross Cutting Concerns : 주요 비즈니스 로직은 아니지만 반복적으로 여러 곳에 쓰이는 부가 기능


  - #### Keyword

    - Aspect : Advice + Pointcut
    - Advice : Target에 제공할 부가 기능을 담고 있는 모듈
    - Target : Advice가 부가 기능을 제공할 대상(Advice가 적용될 비즈니스 로직)
    - JointPoint : Advice가 적용될 위치(적용 시점)
    - PointCut : Target을 지정하는 정규 표현식(적용해야 할 위치)

  - #### Issue Without AOP
    
    - 여러 곳에서 반복적인 코드 작성
    - 코드가 변경될 경우 여러 곳에서 수정
    - 주요 비즈니스와 부가 기능이 한 곳에 섞여 가독성이 떨어진다.

  - #### Proxy Pattern
    - JDK Dynamic Proxy
      - Spring AOP 기본 동작 방식
      - Interface 기반으로 Proxy를 생성해주는 방식
      - Java Reflection을 활용해 동적으로 생성
    - CGLIB Proxy
      - Spring boot AOP 기본 동작 방식
      - 클래스 기반으로 Proxy를 생성해주는 방식

> ### Runtime Weaving

  - Runtime시 JDK Dynamic Proxy or CGLIB Proxy를 활용하여 Proxy를 생성(Target 객체를 새로운 Proxy 객체로 적용하는 과정을 뜻함)
  - Spring AOP는 Proxy를 기반으로 한 Runtime Weaving 방식이다.
  - JDK Dynamic Proxy(Reflection)와 CGLIB(Extends)을 통해 Proxy화 한다.

> ### AspectJ

  - #### Proccess
    - PointCut 표현식을 보고 일치하는 클래스들은 프록시를 만들어서 Bean으로 등록
    - Runtime시 PointCut에 일치하는 메서드들은 Aspect에 정의해 놓은 Advice 로직을 실행 
    - Target의 메서드를 호출

  - #### Example Code

    ```java
    @Aspect
    @Log4j2
    @Component
    public class CustomInterceptor {

      private final LogService logService;

      public CustomInterceptor(LogService logService) {
        this.logService = logService;
      }

      @Around("execution(* com.aop.backend.service.SaleService.*(..))")
      public Object registerSaleLog(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        Object proceed = proceedingJoinPoint.proceed();
        try{
            if (proceed == null) {
                throw new RuntimeException();
            }
            if (proceed instanceof Sale){
                logService.create(SaleLog.of((Sale) proceed));
            }
        }catch (Exception e){
            e.printStackTrace();
            log.error("기록을 입력하지 못하였습니다.", e);
        }
        return proceed;
      }
    }
    ```

<br />

---

