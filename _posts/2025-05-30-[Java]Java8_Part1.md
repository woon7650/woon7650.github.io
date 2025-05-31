---
title: "Refactoring Using Java 8 Functional Interface and Lambda"
excerpt: "[Java] Features supported by Java 8 Part1

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2025-05-30
---

### 들어가면서
  - 현재 프로젝트에서 Java 8이상의 버전을 사용하고 있기에 기존에 개발이 되어있는 오래된 소스들을 리팩토링하고자 합니다.
  - 소스 분리, 패키징, Query 개선과 함께 Java 8에서 제공해주는 기능들을 이용하여 소스 변경을 하고자 합니다.
  - Lambda/Functional Interface, Stream, Optional, Method Reference, ::(Double Colon)에 대하여 확실하게 꼼꼼하게 다시 한번 정리하려고 합니다.

<br />

---

- ### 💡Functional Interface

  - 1개의 Abstract Method(+ n개의 `Default Method/Static Method`)를 가지는 Interface
    - Interface는 기본 구현체를 포함한 Default Method를 포함할 수 있다
    - Lambda Experssion은 Functional Interface로만 사용 가능

      ```java
      @FunctionalInterface
      interface MyFunctionalInterface<T> {
        T printMessage();
        default void defaultCustomMethod(){
          System.out.printl("My Custom Default Method");
        }
        static void staticCustomMethod(){
          System.out.printl("My Custom Static Method");
        }
      }
      ```

  - `@FunctionalInterface`
    - 해당 Interface가 Funtional Interface 조건에 맞는지 검사
    - 해당 Annotation이 없어도 Functional Interace로 동작/사용에는 상관없지만 검증/유지보수를 위해 사용 지향
      - Error 발생(Functional Interface가 아닐 경우) : `Multiple non-overriding abstract methods found in interface` 
    - 위의 예시는 Abstract Method가 하나만 존재하기 때문에 FunctionalInterface

    - 위의 MyFunctionalInterface이기 때문에 Lambda Expression 사용 가능
      ```java
      MyFunctionalInterface<String> myFunctionalInterface = () -> "Override Abstract Method"
      String message = myFunctionalInterface.printMessage();
      //message = "Override Abstract Method"
      myFunctionalInterface.defaultCustomMethod();
      myFunctionalInterface.staticCustomMethod();
      ```


<br />

---

- ### 💡 Java에서 제공하는 Functional Interface

  - Java에서 기본적으로 제공해주는 Functional Interface만 사용해도 왠만한 Lambda식은 만들 수 있기 때문에 직접 Functional Interface를 만드는 경우는 드물다
  - java.util.function
    
    ![image info](/assets/img/FunctionalInterface.png)
    <img src="/assets/img/FunctionalInterface.png" alt="" width="0" height="0"> 


  - #### `Supplier<T>`
    ```java
    @FunctionalInterface
    public interface Supplier<T> {
      T get();
    }
    ```     
    - Parameter X / Return T(공급)
    - Lambda Expression : () -> T 
 
  - #### `Consumer<T> `
    ```java
    @FunctionalInterface
    public interface Consumer<T> {
      void accept(T t);
    }
    ```     
    - Parameter T / Return X(소비)
    - Lambda Expression : T -> void
    - `Consumer<T>의 T는 Reference Type(참조형)만 사용 가능`

  - #### `Predicate<T>`
    ```java
    @FunctionalInterface
    public interface Predicate<T> {
      boolean test(T t);
    }
    ```     
    - Parameter T / Return boolean
    - Lambda Expression : T -> boolean(사실 여부)
    - **Stream filter()**

  - #### `Function<T, R>`
    ```java
    @FunctionalInterface
    public interface Function<T, R> {
      R apply(T t);
    }
    ```
    - Parameter T / Return R
      - T : Type of the input
      - R : Type of the result
    - Lambda Expression : T -> R(함수)
    - 특정 값을 받아서 다른 값으로 반환 
  
  - #### `Comparator<T>`
    ```java
    @FunctionalInterface
    public interface Comparator<T> {
      int compare(T o1, T o2);
    }
    ```
    - Parameter T, T / Return int
    - Lambda Expression : (T, T) -> int(비교)
    - 같은 Type의 인자를 받아서 int로 반환
  
    - #### `Runnable`
    ```java
    @FunctionalInterface
    public interface Comparator<T> {
      public abstract void run();
    }
    ```
    - Parameter X / Return X
    - Lambda Expression : () -> void(실행)
  
    - #### `Callable<V>`
    ```java
    @FunctionalInterface
    public interface Callable<V> {
      V call() throws Exception;
    }
    ```
    - Parameter X / Return T
    - Lambda Expression : () -> T(호출)
  

  - #### 🔁 Generic vs 기본형

    - 해당 Functional Interface들은 Generic 사용
    - Auto-Boxing, Auto-UnBoxing을 통한 비용 소모를 지양한다면 **기본형 특화 함수형 인터페이스** 사용 가능
    - 기본형을 Parameter로 받거나 기본형을 Return 가능

    - Predicate
      - IntPredicate, LongPredicate, DoublePredicate
    - Consumer
      - IntConsumer, LongConsumer, DoubleConsumer
    - Supplier
      - BooleanSupplier, IntSupplier, LongSupplier, DoubleSupplier
    - Function
      - IntToDoubleFunction, IntFunction<R>, ToIntFunction<T>...


<br />

---

- ### 💡 Java에서 제공하는  Bi Functional Interface(Two Parameters)
    
  - #### `BiFunction<T, U>`

    - Parameter (T,U) / Return R
    - Lambda Expression : (T, U) -> R(함수)

      <details>

        <summary>BiFunction.java</summary>
        ```java
        @FunctionalInterface
        public interface BiFunction<T, U, R> {
          R apply(T t, U u);
          default <V> BiFunction<T, U, V> andThen(Function<? super R, ? extends V> after) {
            Objects.requireNonNull(after);
            return (T t, U u) -> after.apply(apply(t, u));
          }
        }
        ```

      </details>

  - #### `BiPredicate<T, U>`

    - Parameter (T,U) / Return boolean
    - Lambda Expression : (T, U) -> boolean(사실 여부)

      <details>

        <summary>BiPredicate.java</summary>

        ```java
        @FunctionalInterface
        public interface BiPredicate<T, U> {
        
          boolean test(T t, U u);
          default BiPredicate<T, U> and(BiPredicate<? super T, ? super U> other) {
            Objects.requireNonNull(other);
            return (T t, U u) -> test(t, u) && other.test(t, u);
          }
          default BiPredicate<T, U> negate() {
            return (T t, U u) -> !test(t, u);
          }
          default BiPredicate<T, U> or(BiPredicate<? super T, ? super U> other) {
            Objects.requireNonNull(other);
            return (T t, U u) -> test(t, u) || other.test(t, u);
          }
        }
        ```

      </details>

  - #### `BiConsumer<T, U>`

    - Parameter (T,U) / Return void
    - Lambda Expression : (T, U) -> void(소비)

      <details>

        <summary> BiConsumer.java </summary>

        ```java
        @FunctionalInterface
        public interface BiConsumer<T, U> {
        
          void accept(T t, U u);
          default BiConsumer<T, U> andThen(BiConsumer<? super T, ? super U> after) {
            Objects.requireNonNull(after);
              return (l, r) -> {
                accept(l, r);
                after.accept(l, r);
              };
          }
        }
        ```

      </details>



<br />

---

- ### 💡Lambda Expression


  - 익명 함수(Anonymous Function)의 한 종류(Method를 하나의 식으로 표현)
    - Function을 하나의 Expression으로 표현한 것으로 람다식으로 표현하면 Method의 이름이 없다
  - 익명 함수들은 모두 1급 개체(변수처럼 사용이 가능하며 매개 변수로 전달이 가능)
  - 1급 객체 조건
    - 변수나 데이터에 할당할 수 있어야 한다
    - 객체의 인자로 넘길 수 있어야 한다
    - 객체의 리턴값으로 리턴할 수 있어야 한다

  - #### Usage & Condition

    - `Functional Interface 필요`
    - 사용 형태 : `(Parameter...) -> {실행문};`
    - `Type 생략 가능`(Functional Interface에 이미 Type이 명시되어 있음)
    - Parameter가 1개인 경우 `소괄호() 생략 가능`
    - logic 부분에 실행문이 1개 & return문 일 경우 -> `중괄호{}와 return 생략 가능`

  - #### Pros
    - 코드 간결성(Lambda Expression을 사용하면 코드를 훨씬 간결하게 표현이 가능)
    - 생산성 증가(함수를 만드는 과정 생략 가능)
    - 병렬 처리(Multi-Thread를 사용하여 병렬 처리 가능)

  - #### Cons
    - 함수의 재사용이 어려움
    - 디버깅의 어려움



<br />

---



- ### ✅Conclusion
  - 본 포스트에서는 Functional Interface와 Lambda에 대해서 다음 포스트에서 Stream, Optional, Method Reference를 다루고 실제 상황에서 lambda의 적용에 대해서 알아보고자 합니다.
  실제 사용과 활용도 중요하지만 실제로 눈에 보이지는 않지만 사용되는 부분들과 과정에 대해 명확히 알면 사용 시에 잘못된 부분의 파악과 효율적인 활용이 더욱 부각될 것이라고 생각합니다.


<br />

---


- ### Reference

  - https://dwaejinho.tistory.com/entry/Java-Lambda-Stream-%EB%8F%84%EC%9E%85-%EB%B0%B0%EA%B2%BD%EA%B3%BC-%EC%9B%90%EB%A6%AC-%ED%8C%8C%ED%95%B4%EC%B9%98%EA%B8%B0
  - https://sunrise-min.tistory.com/entry/Java-8-%EB%9E%8C%EB%8B%A4Lambda-%EC%8A%A4%ED%8A%B8%EB%A6%BCStream-double-colon
  - https://bcp0109.tistory.com/313
  - https://medium.com/@logishudson0218/understanding-for-java-1-8-lambda-%EB%9E%8C%EB%8B%A4%EC%8B%9D-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-43bcc491519a
  - https://velog.io/@snake7667/Java-Java8-Lambda%EC%99%80-%ED%95%A8%EC%88%98%ED%98%95-%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4