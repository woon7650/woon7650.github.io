---
title: "Refactoring Using Java 8 Stream API, Method Reference, Optional"
excerpt: "[Java] Features supported by Java 8 Part2"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2025-06-09
---

### 들어가면서
  - 저번 포스팅의 Lambda/Functional Interface에 이어서 Java 8이후 제공해주는 Stream, Optional, Method Reference, ::(Double Colon)에 대하여 심화 학습 후 정리해보고자 합니다.

<br />

---

- ### 💡Method Reference

  - Lambda Expression을 더 간단하게 표현하는 방법
  - Method Reference에는 불필요한 매개변수를 제거하는 등 생략이 많이 이루어지기 때문에 `Method의 parameter과 return type`에 대해 알고 있어야 한다
  - `Double Colon(::)` : Method명/Class 또는 Method명/객체명을 분리합니다
    - Class::Method
    - Instance::Method
    - Class::new

    ```java
    //System.out println()
    Consumer<String> printWithLambda = name -> System.out.println(name);
    Consumer<String> printWithMethodReference = System.out::println;

    //String length()
    Function<String, Integer> lengthWithLambda = string -> string.length();
    Function<String, Integer> lengthWithMethodReference = String::length;

    //Object equals()
    BiFunction<String, String, Boolean> equalWithLambda = (str1, str2) -> str1.equals(str2);
    BiFunction<String, String, Boolean> equalWithMethodReference = Object::equals;

    //Math max()
    BinaryOperator<Integer> maxWithLambda = (num1, num2) -> Math.max(num1, num2);
    BinaryOperator<Integer> maxWithMethodReference = Math::max;
    ```

  - #### Example Custom Class

    ```java
    @Getter
    @Setter
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public class Menu {
    
      private String name;
      private Integer price;

      public static Menu of(String name, Integer price) {
        return Menu.builder()
                    .name(name)
                    .price(price)
                    .build();
          
      }
    }

    ```

  - #### Static Method Reference
    
    - `Static Method Reference -> Class::Method(static)`
    - Menu Class의 static Method를 참조해서 Instance 생성

      ```java
      BiFunction<String, Integer, Menu> createMenuMethodReference = Menu::of;
      Menu cafeMenu = createMenuMethodReference.apply("coffee", 5000);
      ```


  - #### Instance Method Reference
      
    - `Instance Method Reference -> Instance::Method(public)`
    - 특정 Instance의 Method를 참조하여 값을 반환

      ```java
      BiFunction<String, Integer, Menu> createMenuMethodReference = Menu::of;
      Menu cafeMenu = createMenuMethodReference.apply("coffee", 5000);

      Supplier<String> name = cafeMenu::getName;
      Supplier<Integer> price = cafeMenu::getPrice;

      System.out.println(name + " 가격 : " + price);
      ```

  - #### Constructor Method Reference
      
    - `Constructor Method Reference -> Class::new`
    - 기본 Constructor 참조를 통한 Instance 생성은 get()을 해야 Instance가 생성된다
    - Constructor 참조 -> apply() parameter를 통해서 Instance를 생성할 수 있다

      ```java
      Supplier<Menu> defaultConstructorMenu = Menu::new;
      Menu defaultMenu = defaultConstructorMenu.get();

      Supplier<Menu> customConstructorMenu = Menu::new;
      Menu cafeMenu = customConstructorMenu.apply("coffee", 5000);
      ```

- ### 💡 Stream API

  - Stream API는 Collection을 Functional Interface를 통해 직관적으로 처리할 수 있도록 제공하는 API
  - Java 8에 추가되어 대용량 데이터를 다룰 경우 사용하는 Array나 Collection의 비효율적인 부분을 개선하기 위해 탄생한 방법
  - 기존의 반복문 방식과 다르게 Stream API는 데이터 처리 과정을 메서드 체이닝 방식으로 표현할 수 있어 가독성이 크게 향상된다
  
  - **반복문(for, while..)과 Stream의 성능 차이**
    > 일반적으로 Stream은 반복문에 비해 성능이 느리다. 기존 반복문은 Compiler나 JVM, Optimizer 등에 의해 오랜 시간 최적화가 되어 왔기 때문에 Primitive Type 연산은 성능차이가 크게 난다. 하지만 Wrapper Type의 Collection을 사용하도록 강제한다면 성능 차이는 20% 정도로 확연하게 줄어든다.

  - #### Features
    - 불변성
      - 원본 데이터의 복사 없이 원본 데이터는 유지 하면서 새로운 Collection을 만들어 낼 수 있다
        - Stream은 데이터를 읽는 작업 -> Stream 연산을 통한 원본 데이터는 변경되지 않는다
        - Usage : `Data의 깊은 복사 없이 원본 Data의 변경 없이 새로운 Data를 만들어내야 하는 경우`
    - 일회성
      - Stream은 데이터를 모두 읽고 나면 사라진다
      - Stream 객체를 재사용 했을 경우 : Runtime Error 발생(`java.lang.IllegalStateException: stream has already been operated upon or closed`)
    - 지연 처리(Lazy Evaluation)
      - Stream pipeline : 중간/종료 연산이 연속적으로 연결된 구조
      - 중간 연산은 실제로 처리되지 않고 종료 연산이 실행될 때 한번에 처리
      - `filter(), map()`은 중간 연산으로 지연 처리되며 `collect()` 종료 연산이 실행될 때 실제로 모든 연산이 수행
    - 병렬 처리
      - `parallelStream()`를 사용하여 생성 / 기존 Stream에서 `parallel()`를 사용하여 병렬 Stream으로 변환
      - 내부적으로 `ForkJoinPool`을 통해 MultiThread 환경에서 data를 처리
      - `forEachOrdered()`를 사용하여 순서를 보장하는 병렬 처리를 할 수 있다
      - 병렬 Stream을 사용한 후에 `sequential()`를 사용하여 다시 순차 Stream으로 변환할 수 있다

    - 내부 반복
      - Collection은 `외부 반복(반복문, forEach)`을 통해 작업 / Stream은 `내부 반복(Stream의 연산)`으로 작업

  - #### Process
    - Source : Collection(`List`, `Set`..), Array, File I/O..
      - `Source -> [중간 연산] -> [중간 연산] -> ... -> [최종 연산]`
    - `Intermediate Operations(중간 연산)`
      - Data를 변환하거나 Filtering하는 작업을 수행(이어서 Stream 연산이 가능)
      - `항상 지연 처리(Lazy Evaluation)로 실행, 최종 연산 실행까지 실제로 수행되지 않음`
      - 여러 중간 연산을 연결하여 사용, 각 연산의 결과가 다음 연산에 입력
      - Return Type : Stream<T>
    - `Terminal Operations(최종 연산)`
      - Stream을 소비, 데이터를 실제로 처리하여 결과를 return(더 이상 연산이 불가능)
      - 해당 연산이 호출되면 Stream pipeline 전체가 실행
      - Stream은 소모되며 한 번 최종 연산이 호출된 Stream은 다시 사용할 수 없다
      - Return Type : **void**, **Optional<T>**, **List<T>**...

  - #### Source(Stream generate)
    - Collection : .stream()
    - Array : Arrays.stream(array명)
    - File I/O : Files.lines(filePath)
      - `java.nio.file`를 사용하여 각 줄을 읽고 처리할 수 있다
    - Number : range(), rangeClose()
      - Java에서 `IntStream, LongStream, DoubleStream`를 숫자형 Stream을 제공, Primitive Type의 숫자를 효율적으로 처리 가능

  - #### Intermediate Operation


    ![image info](/assets/img/Stream1.png)
    <img src="/assets/img/Stream1.png" alt="" width="0" height="0"> 


  - #### Terminal Operation

    ![image info](/assets/img/Stream2.png)
    <img src="/assets/img/Stream2.png" alt="" width="0" height="0"> 

  - #### Stream Performance
    - Stream 성능은 Data 구조에 따라 다르게 나타난다
      - `ArrayList` : 랜덤 접근이 가능한 데이터 구조에서는 Stream 성능이 좋음
      - `LinkedList` : 순차 접근이 필요한 구조에서는 성능이 떨어질 수 있음


  - ### Collectors Utility

    - Collector : Stream 요소를 어떤 식으로 도출할지 지정한다
    - Stream에서 .collect()를 호출하면 `Collector가 Stream Element에 Reducing 연산을 수행`하여 필요한 데이터 구조로 간단하게 도출한다
    - Collectors에서 제공하는 Method 기능
      - Stream Element를 하나의 값으로 Reducing하고 요약
      - Stream Element 그룹화
      - Stream Element 분할

    - Collectors 클래스에서 제공하는 Static Factory Method
      ![image info](/assets/img/Stream3.png)
      <img src="/assets/img/Stream3.png" alt="" width="0" height="0"> 



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



- ### Reference

  - https://countryxide.tistory.com/127
  - https://www.daleseo.com/java8-optional-effective/
  - https://gom20.tistory.com/217
  - https://jong99.tistory.com/150
  - https://jong99.tistory.com/177
  - https://veneas.tistory.com/entry/java-%EC%9E%90%EB%B0%94-8-%EB%A9%94%EC%86%8C%EB%93%9C-%EB%A0%88%ED%8D%BC%EB%9F%B0%EC%8A%A4-Method-Reference
  - https://pamyferret.tistory.com/43
  - https://velog.io/@dankj1991/Java-Stream-API
  - https://velog.io/@songsunkook/Java-Stream-API-feat.-%EC%B5%9C%EC%A0%81%ED%99%94
  - https://devbksheen.tistory.com/entry/%EB%AA%A8%EB%8D%98-%EC%9E%90%EB%B0%94-%EC%BB%AC%EB%A0%89%ED%84%B0Collector%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80