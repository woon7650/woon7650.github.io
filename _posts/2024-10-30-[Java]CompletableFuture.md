---
title: "Asynchronous Programming With CompletableFuture"
excerpt: "[Java] About CompletableFuture & Limit of Future"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2024-10-30
---

##### RELATED POST
- [Spring Async](blog/Spring-Async/)
- [Spring AsyncNonBlocking](blog/Spring-AsyncNonBlocking/)


> ### 들어가면서

  > CompletableFuture is part of Java's java. util. concurrent package and provides a way to write asynchronous code by representing a future result that will eventually appear. It lets us perform operations like calculation, transformation, and action on the result without blocking the main thread.

  > CompleableFuture는 Java의 java.util. 동시 패키지의 일부이며, 나중에 나타날 미래 결과를 표현하여 비동기 코드를 작성하는 방법을 제공합니다. 이를 통해 메인 스레드를 차단하지 않고도 결과에 대한 계산, 변환 및 작업과 같은 작업을 수행할 수 있습니다.

<br />

---

> ### Future

  - Java5에 추가된 Future는 비동기 작업의 결과를 나타내는 Interface
  - 작업이 아직 완료되지 않았더라도 결과에 접근할 수 있는 방법을 제공

  - #### Example

    ```java
    public static void main(String[] args) throws ExecutionException, InterruptedException {
      ExecutorService executorService = Executors.newFixedThreadPool(4);
      Future<String> future = executorService.submit(() -> "Complete");

      future.get(); 
      //get을 할 때까지 block
    }
    ```

    - get()은 Blocking Call이라서 future.get() 이후의 작업은 비동기 처리 결과를 얻고 나서 진행된다.

  - #### Limitation

    - 비동기 작업 실행
    - 작업 콜백
    - 작업 조합
    - 예외 처리

  - #### Solution
    - **Future**을 이용해서 처리 하기 힘든 비동기 프로그래밍을 **CompletableFuture**로 해결해보자 합니다.

<br />

---


> ### Asynchronous Job Execution in CompletableFuture

  - CompleteableFuture가 제공하는 비동기 작업 실행
  - runAsync
    - 반환 값이 없는 경우
    - 비동기로 작업 실행 call
  - supplyAsync
    - 반환 값이 있는 경우
    - 비동기로 작업 실행 call

    ```java
    public class CompletableFuture<T> implements Future<T>, CompletionStage<T> {

      //supplyAsync
      public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier) {
          return asyncSupplyStage(ASYNC_POOL, supplier);
      }
      public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor) {
          return asyncSupplyStage(screenExecutor(executor), supplier);
      }

      //runAsync
      public static CompletableFuture<Void> runAsync(Runnable runnable) {
        return asyncRunStage(ASYNC_POOL, runnable);
      }
      public static CompletableFuture<Void> runAsync(Runnable runnable, Executor executor) {
          return asyncRunStage(screenExecutor(executor), runnable);
      }

    }
    ```


<br />

---

> ### Callback in CompletableFuture

  - thenApply
    - 반환 값을 받아서 다른 값을 반환함
    - parameter : 함수형 인터페이스 Function
  - thenAccept
    - 반환 값을 받아 처리하고 값을 반환하지 않음
    - parameter : 함수형 인터페이스 Consumer
  - thenRun
    - 반환 값을 받지 않고 다른 작업을 실행함
    - parameter : 함수형 인터페이스 Runnable


    ```java
    public class CompletableFuture<T> implements Future<T>, CompletionStage<T> {

      public <U> CompletableFuture<U> thenApply(Function<? super T,? extends U> fn) {
        return uniApplyStage(null, fn);
      }

      public CompletableFuture<Void> thenAccept(Consumer<? super T> action) {
        return uniAcceptStage(null, action);
      }

      public CompletableFuture<Void> thenRun(Runnable action) {
        return uniRunStage(null, action);
      }
    }
    ```

<br />

---

> ### Combination in CompletableFuture

  - thenCompose
    - 두 작업이 이어서 실행하도록 조합하고 압선 작업의 결과를 받아서 Callback 실행
    - parameter : 함수형 인터페이스 Function
  - thenCombine
    - 두 작업을 독립적으로 실행하고 모두 완료되었을 때 Callback 실행
    - parameter : 함수형 인터페이스 Function
  - allOf
    - 여러 작업들을 동시에 실행하고 모든 작업 결과에 Callback 실행
  - anyOf
    - 여러 작업들 중에서 가장 빨리 끝난 하나의 결과에 Callback 실행


    ```java
    public class CompletableFuture<T> implements Future<T>, CompletionStage<T> {

      public <U> CompletableFuture<U> thenCompose(Function<? super T, ? extends CompletionStage<U>> fn) {
        return uniComposeStage(null, fn);
      }

      public <U,V> CompletableFuture<V> thenCombine(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn) {
        return biApplyStage(null, other, fn);
      } 

      public static CompletableFuture<Void> allOf(CompletableFuture<?>... cfs) {
        return andTree(cfs, 0, cfs.length - 1);
      }

      public static CompletableFuture<Object> anyOf(CompletableFuture<?>... cfs) {
        int n; Object r;
        if ((n = cfs.length) <= 1)
          return (n == 0) ? new CompletableFuture<Object>() : uniCopyStage(cfs[0]);
        for (CompletableFuture<?> cf : cfs)
          if ((r = cf.result) != null)
            return new CompletableFuture<Object>(encodeRelay(r));
        cfs = cfs.clone();
        CompletableFuture<Object> d = new CompletableFuture<>();
        for (CompletableFuture<?> cf : cfs)
          cf.unipush(new AnyOf(d, cf, cfs));
        if (d.result != null)
          for (int i = 0, len = cfs.length; i < len; i++)
            if (cfs[i].result != null)
              for (i++; i < len; i++)
                if (cfs[i].result == null)
                  cfs[i].cleanStack();
        return d;
      }
    }
    ```

<br />

---



> ### ExceptionHandling in CompletableFuture

  - exceptionally
    - 발생한 error를 받아서 에외 처리 가능
    - parameter : 함수형 인터페이스 Function

  - handle, handleAsync
    - 결과값, error를 받아서 에러가 발생한 경우 아닌 경우 모두 처리 가능
    - parameter : 함수형 인터페이스 BiFunction

    ```java
    public class CompletableFuture<T> implements Future<T>, CompletionStage<T> {

      public CompletableFuture<T> exceptionally(Function<Throwable, ? extends T> fn) {
        return uniExceptionallyStage(null, fn);
      }

      public <U> CompletableFuture<U> handle(BiFunction<? super T, Throwable, ? extends U> fn) {
        return uniHandleStage(null, fn);
      }
      public <U> CompletableFuture<U> handleAsync(BiFunction<? super T, Throwable, ? extends U> fn) {
        return uniHandleStage(defaultExecutor(), fn);
      }

      public <U> CompletableFuture<U> handleAsync(BiFunction<? super T, Throwable, ? extends U> fn, Executor executor) {
        return uniHandleStage(screenExecutor(executor), fn);
      }
    }
    ```

<br />

---

### Reference

- https://wbluke.tistory.com/50
- https://velog.io/@brucehan/CompletableFuture-%EC%A0%95%EB%A6%AC
- https://sh970901.tistory.com/139
- https://mangkyu.tistory.com/263
- https://velog.io/@suyeon-jin/JAVA-CompletableFuture