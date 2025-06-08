---
title: "Refactoring Using Java 8 Stream, Method Reference, Optional"
excerpt: "[Java] Features supported by Java 8 Part2"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2025-05-31
---

### 들어가면서
  - 현재 프로젝트에서 Java 8이상의 버전을 사용하고 있기에 기존에 개발이 되어있는 오래된 소스들을 리팩토링하고자 합니다.
  - 소스 분리, 패키징, Query 개선과 함께 Java 8에서 제공해주는 기능들을 이용하여 소스 변경을 하고자 합니다.
  - Lambda/Functional Interface, Stream, Optional, Method Reference, ::(Double Colon)에 대하여 확실하게 꼼꼼하게 다시 한번 정리하려고 합니다.

<br />

---

- ### 💡Method Reference

  - Lambda Expression을 더 간단하게 표현하는 방법
  - 불필요한 매개변수를 제거할 수 있다
  - Reference에는 생략이 많이 이루어지기 때문에 Method의 parameter과 return type에 대해 알고 있어야 한다

    ```java
    // 받은 값 출력
    Consumer<String> printName1 = name -> System.out.println(name);
    Consumer<String> printName2 = System.out::println;

    // 문자열 수 반환
    Function<String, Integer> lengthStr1 = string -> string.length();
    Function<String, Integer> lengthStr2 = String::length;

    // 문자열 비교
    BiFunction<String, String, Boolean> equalCheck1 = (str1, str2) -> str1.equals(str2);
    BiFunction<String, String, Boolean> equalCheck2 = Object::equals;

    // 숫자 비교(큰 수 반환)
    BinaryOperator<Integer> maxNum1 = (num1, num2) -> Math.max(num1, num2);
    BinaryOperator<Integer> maxNum2 = Math::max;
    ```

  - #### Static Method Reference
    
    - `static method가 포함된 class명::static method명`

      ```java
      num -> Math.abs(num)

      Math::abs
      ```

  - #### Arbitary Obejct Method Reference
      
    - `Type::method명`

      ```java
      str -> str.toLowerCase()

      String::LowerCase()
      ```

  - #### Generated Instance Method Reference
      
    - `method가 포함된 Instance명::instance의 method명`

      ```java
      CustomComparator customComparator = new CustomComparator();

      (a, b) -> customComparator.compare(a,b)

      customComparator::compare
      ```

  - https://veneas.tistory.com/entry/java-%EC%9E%90%EB%B0%94-8-%EB%A9%94%EC%86%8C%EB%93%9C-%EB%A0%88%ED%8D%BC%EB%9F%B0%EC%8A%A4-Method-Reference 참조




- ### 💡 Stream API

  - Java 8에 추가되어 대용량 데이터를 다룰 경우 사용하는 Array나 Collection의 비효율적인 부분을 개선하기 위해 탄생한 방법
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


  - 원본 데이터의 복사 없이 원본 데이터는 유지 하면서 새로운 Collection을 만들어 낼 수 있다
  - Stream은 데이터를 모두 읽고나며녀 사라지는 1회용
    - Stream 객체를 재사용 했을 경우
    - `java.lang.IllegalStateException: stream has already been operated upon or closed`

  - #### Features
    - Collection은 외부 반복을 통해 작업 / Stream은 내부 반복으로 작업
    - Stream은 재사용이 불가능하며 한 번만 사용이 가능
    - Stream은 원본 데이터를 변경하지 않는다
    - Stream 연산은 Filter-Map 기반의 API를 사용해 Lazy 연산을 통해 성능을 최적화(인자 하나씩 조건을 순회하여 불필요한 연산 수행을 방지)
    - parallelStream()를 통한 병렬 처리를 지원(MultiThread로 처리 가능)

  - #### Intermediate Operation
    - Stream 필터링: filter(), distinct()
    - Stream 변환: map(), flatMap()
    - Stream 제한: limit(), skip()
    - Stream 정렬: sorted()
    - Stream 연산결과 확인: peek()

  - #### Terminal Operation
    - Element 출력 : forEach()
    - Element 소모 : reduce()
    - Element 검색 : findFirst(), findAny()
    - Element 검사 : anyMatch(), allMatch(), noneMatch()
    - Element 통계 : count(), min(), max()
    - Element 연산 : sum(), average()
    - Element 수집 : collect()

  
  - https://pamyferret.tistory.com/43 참고


- ### 💡 Optional
    
  - Java 8 이후로 추가된 Wrapper class
  - NullPointException을 간단하게 방지할 수 있다
  
  - isPresent()로 null의 여부를 파악 -> get()으로 Optional 객체에서 원래의 객체값을 가져옴


  - #### Primitive Type Optional Class

    - OptionalInt(int getAsInt())
    - OptionalLong(long getAsLong())
    - OptionalDouble(double getAsDouble())

  - #### Method
    - T get()
    - boolean isPresent()
    - Optional of(T value)
    - Optional ofNullable(T value)


    - T orElse(T other)
     - 저장된 값이 존재하면 그 값을 반환하고, 값이 존재하지 않으면 인수로 전달된 값을 반환
    - T orElseGet(Supplier<? extends T> other)
      - 저장된 값이 존재하면 그 값을 반환하고, 값이 존재하지 않으면 인수로 전달된 람다 표현식의 결괏값을 반환함
    - T orElseThrow(Supplier<? extends X> exceptionSupplier)
      - 저장된 값이 존재하면 그 값을 반환하고, 값이 존재하지 않으면 인수로 전달된 예외를 발생시킴


  
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

- ### Reference

  - https://countryxide.tistory.com/127
  - https://www.daleseo.com/java8-optional-effective/
  - https://gom20.tistory.com/217
  - https://jong99.tistory.com/150
  - https://jong99.tistory.com/177
  - https://veneas.tistory.com/entry/java-%EC%9E%90%EB%B0%94-8-%EB%A9%94%EC%86%8C%EB%93%9C-%EB%A0%88%ED%8D%BC%EB%9F%B0%EC%8A%A4-Method-Reference
  - https://pamyferret.tistory.com/43