---
title: "Java8 Stream and Lambda"
excerpt: "[Java] Usage of Stream, Lambda of Java 8"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2025-05-20
---


- ### 💡Lambda Expression

  - 익명 함수(Anonymous Function)를 지칭하는 용어
    - Function을 하나의 Expression으로 표현한 것으로 람다식으로 표현하면 Method의 이름이 없다
  - 익명 함수들은 모두 1급 개체(변수처럼 사용이 가능하며 매개 변수로 전달이 가능)
    - Stream API의 매개 변수로 전달이 가능

  - 1급 객체
    - 변수나 데이터에 할당할 수 있어야 한다
    - 객체의 인자로 넘길 수 있어야 한다
    - 객체의 리턴값으로 리턴할 수 있어야 한다

  - #### Feature
    - 코드 간결성
    - 병렬 처리(Multi-Thread를 사용하여 병렬 처리 가능)

- ### 💡Functional Interface

  - Java의 Lambda Expression이 Functional Interface를 반환
  - `@FunctionalInterface`
    - 구현해야 하는 추상 메소드가 하나만 정의된 인터페이스
  
  - Java에서 제공하는 Functional Interface
    - Supplier<T> 
      - Parameter x, Return Type T **(() -> T)**         
    - Consumer<T> 
      - Parameter T, Return x **(T -> {})**        
    - Function<T, R>
      - Parameter T, Return R **(T -> R)**    
      - T : Type of the input
      - R : Type of the result      
    - Predicate<T> : filter()는 매개변수로 predicate를 받는다
      - Parameter T, Return true/false **(T -> boolean)** 
    - BiFunction<T, U, R>
      - Parameter T/U, Return R **((T,U) -> R)** 
      - T : Type of the first argument
      - U : Type of the second arguement
      - R : Type of the result

    - Runnable : Parameter x, Return x **(() -> {})**       
    - UnaryOperator<T> : Parameter T, Return T **(T -> T)** 
    - BinaryOperator<T> : Parameter T/T, Return T **((T,T) -> T)**

- ### 💡Stream API

  - Collection을 선언형으로 처리할 수 있게 해주는 API
  - `Source -> [중간 연산] -> [중간 연산] -> ... -> [최종 연산]`
  - 구조
    - Source : Collection(`List`, `Set`..)
    - Intermediate Operations
      - 처리 단계로 stream을 반환
      - 지연 실행 -> 최종 연산이 있어야 실행
      - Return Type : Stream<T>
    - Terminal Operations
      - 실행 단계로 stream을 소비하고 결과를 생성
      - Stream을 소비하며 종료하고 실제 결과를 생성
      - Return Type : **void**, **Optional<T>**, **List<T>**...

  - #### Intermediate Operations(중간 연산)
    - filter(Predicate)
    - map(Function)      
    - flatMap(Function) 
    - distinct()          
    - sorted()         
    - sorted(Comparator)
    - limit(n)      
    - skip(n)     
    - peek(Consumer)  

  - #### Terminal Operations(최종)
    - forEach(Consumer)
    - collect(Collectors)  
    - count()   
    - reduce()             
    - toArray()           
    - anyMatch() / allMatch()
    - findFirst() / findAny()

- ### 💡Functional Interface <-> Stream API
  - 함수형 인터페이스는 Stream 연산의 핵심 입력 규칙이다
  - Stream의 중간 연산은 대부분 Function, Predicate 기반
  - Stream의 최종 연산은 Consumer, BinaryOperator 등과 밀접

  
<br />

---

- ### Java 8 method References, Double Colon(::), operator

  - ClassName::staticMethodName(Reference -> static method)
  - object::instanceMethodName(Reference -> instance method of a particular object)
  - ContainingType::methodName(Reference -> instance method of an arbitrary object of a particular type)
  - ClassName::new(Reference -> constructor)

- ### Collectors Utility
  - Collectors.toList()
  - Collectors.toSet()
  - Collectors.joining(", ")
  - Collectors.groupingBy(...)
  - Collectors.partitioningBy(...)
  - Collectors.averagingInt(...)



- ### Usage

  - #### 적합한 상황

    - 대량의 데이터를 **필터링, 매핑, 집계**해야 할 때
    - **이벤트 처리 / 콜백** 구현이 필요한 경우
    - **병렬 처리**로 성능을 높이고자 할 때
    - **코드 가독성**을 높이고자 할 때

  - #### 피해야 할 상황

    - 스트림 내부에서 **예외 처리가 많은 경우**
    - **복잡한 상태 변경 로직**이 필요한 경우
    - **디버깅을 자주** 해야 하는 경우
    - **짧은 컬렉션**에서 성능이 중요한 경우

<br />

---

- ### ✅Conclusion
  - 개발도 중요하겠지만 어플리케이션이 돌아가는 과정과 관련된 기본적인 개념에 대해서 한 번씩 정리하는 과정을 통해서 다음에 비슷한 jar와 dll 관련된 이슈가 생기더라도 빠르게 대처할 수 있을 것 같습니다.  


---

- ### Reference

  - https://dwaejinho.tistory.com/entry/Java-Lambda-Stream-%EB%8F%84%EC%9E%85-%EB%B0%B0%EA%B2%BD%EA%B3%BC-%EC%9B%90%EB%A6%AC-%ED%8C%8C%ED%95%B4%EC%B9%98%EA%B8%B0
  - https://sunrise-min.tistory.com/entry/Java-8-%EB%9E%8C%EB%8B%A4Lambda-%EC%8A%A4%ED%8A%B8%EB%A6%BCStream-double-colon