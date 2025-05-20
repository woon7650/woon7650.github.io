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

  - 익명 함수(Anonymous Function)를 작성하는 간결한 방법
  - 코드의 가독성을 높이고 불필요한 코드 작성을 줄일 수 있음




- ### 💡Functional Interface

  - `Runnable` : 매개변수 없음, 반환 없음 `() -> {}`          
  - `Supplier<T>` : 매개변수 없음, T 반환 `() -> T`         
  - `Consumer<T>` : T 입력, 반환 없음 `T -> {}`         
  - `Function<T, R>` : T 입력, R 반환 `T -> R`           
  - `Predicate<T>` : T 입력, boolean 반환 `T -> true/false`  
  - `BiFunction<T, U, R>` : 두 입력, R 반환 `(T, U) -> R`         
  - `UnaryOperator<T>` : T 입력, T 반환 (Function 확장) `T -> T`             
  - `BinaryOperator<T>` : 두 T 입력, T 반환 (BiFunction 확장) `(T, T) -> T`      

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
    - `filter(Predicate)`
    - `map(Function)`       
    - `flatMap(Function)` 
    - `distinct()`           
    - `sorted()`            
    - `sorted(Comparator)`
    - `limit(n)`          
    - `skip(n)`           
    - `peek(Consumer)`    

  - #### Terminal Operations(최종)
    - `forEach(Consumer)`      
    - `collect(Collectors)`      
    - `count()`               
    - `reduce()`               
    - `toArray()`             
    - `anyMatch() / allMatch()`
    - `findFirst() / findAny()`

- ### 💡Functional Interface <-> Stream API
  - 함수형 인터페이스는 Stream 연산의 핵심 입력 규칙이다.
  - Stream의 중간 연산은 대부분 Function, Predicate 기반
  - Stream의 최종 연산은 Consumer, BinaryOperator 등과 밀접

- ### Java 8(Lambda + Stream) vs before Java 8
  - 코드 길이 : 간결하고 선언적
  - 가독성 : 높음
  - 병렬 처리 : `parallelStream()`로 가능
  - 디버깅 : 다소 어려움

<br />

---


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