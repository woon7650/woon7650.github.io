---
title:  "Interceptor에 대하여"
excerpt: "About Interceptor..."

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
## Interceptor의 종류

> *preHandle()*

  - Controller 호출 전에 실행됩니다.
  - Controller 이전에 처리해야하는 전처리 작업, 요청 데이터를 변경, 추가해야 하는 경우에 사용합니다.
  - return 타입이 true이면 다음 단계로 진행, false이면 작업을 중단하여 이후의 작업은 진행되지 않습니다.
  
```java
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception{
    return true;
}
```

> *postHandle()*

  - Controller 호출 후에 실행됩니다.
  - Controller 이후에 처리해야하는 후처리 작업을 합니다.
  - return 타입은 controller가 반환하는 ModelAndView 타입의 정보입니다.

```java
public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable ModelAndView modelAndView) throws Exception{

}
```

> *afterCompletion()*

  - 모든 View에서 최종 결과를 생성하는 일을 포함해 모든 작업이 완료된 후에 실행 됩니다.
  - 요청 처리 중에 사용한 리소스를 반환할 때 사욯아기에 적합합니다.

```java
public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable Exception ex) throws Exception{

}
```

---

## Spring에서의 Interceptor 지원
- HandlerInterceptor interface
- HandlerInterceptorAdaptor abstract class