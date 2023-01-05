---
title:  "[Backend]Interceptor"
excerpt: "Interceptor"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2022-01-24
---

## Interceptor??

- 개념
    - client가 sever로 특정 URI를 요청시 Controller로 들어오는 Request객체를 handler에 도달하기 전에 가로채는 Module입니다.
    - client에서 온 요청이 handler로 가기전에 추가 작업이나 로직을 수행할 수 있습니다.
- 특징
    - 인증/인가 등과 같은 공통 작업을 하는 코드의 재사용성을 증가시킵니다.
    - API호출에 대한 Logging 또는 검사에 용이합니다.
    - Controller로 넘겨주는 데이터의 가공에 적합합니다.
    - Spring Container에서 동작합니다.
    - Filter를 거치고 Dispatcher Servlet이 요청을 받은 이후에 동작합니다.
    - HttpServletRequest, HttpServletResponse 객체를 제공 받고 Controller로 넘겨주기 위한 data를 가공합니다.

---

## Spring에서의 Interceptor

> pom.xml에 spring-web Dependency의 존재 여부를 확인합니다.

> HandlerInterceptor interface, HandlerInterceptorAdaptor를 구현하는 Interceptor Class를 구현합니다.

```java
public class MyInterceptor implements HandlerInterceptor{
//HandlerInterceptor 상속
}
```

```java
public class MyInterceptor extends HandlerInterceptorAdapter{
//HandlerInterceptorAdapter 상속
}
```

> `<mvc:exclude-mapping path=""/>` 를 이용한 Interceptor 경로 제외합니다.

```xml
<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/**"/>
        <!--모든 URL 요청이 interceptor를 거친다-->
        <mvc:exclude-mapping path="/example.do"/>
        <!---interceptor를 거치지 않은 URL 요청 정의-->
        <bean class="controller.exampleInterceptor"></bean>
        <!--interceptor controller bean 정의-->
    </mvc:interceptor>
</mvc:interceptors>
```

## Interceptor의 종류

> *preHandle()*

  - Controller 호출 전에 실행됩니다.
  - Controller 이전에 처리해야하는 전처리 작업, 요청 데이터를 변경, 추가해야 하는 경우에 사용합니다.
  - return 타입이 true이면 다음 단계로 진행, false이면 작업을 중단하여 이후의 작업은 진행되지 않습니다.
  
```java
@Override
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception{
    return true;
}
```

> *postHandle()*

  - Controller 호출 후에 실행됩니다.
  - Controller 이후에 처리해야하는 후처리 작업을 합니다.
  - return 타입은 controller가 반환하는 ModelAndView 타입의 정보입니다.
  - preHandle() return이 false일 경우 실행되지 않습니다.

```java
@Override
public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable ModelAndView modelAndView) throws Exception{

}
```

> *afterCompletion()*

  - View rendering 완료 후 client에게 response를 전달하기 전에 추가적인 작업이 가능합니다.
  - 요청 처리 중에 사용한 리소스를 반환할 때 사용하기에 적합합니다.
  - preHandle() return이 false일 경우 실행되지 않습니다.

```java
@Override
public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable Exception ex) throws Exception{

}
```
---

## Interceptor 동작 순서

- preHandler
- request 처리
- postHandler
- view rendering
- aftercompletion

---
