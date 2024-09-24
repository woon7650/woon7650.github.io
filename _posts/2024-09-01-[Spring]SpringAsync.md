---
title: "Asynchronous Processing Technology Provided By Spring"
excerpt: "[Spring] Spring에서 제공하는 비동기 처리 기술"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2024-09-01
---

##### RELATED POST
-  [ThreadPool & ExecutorService](blog/Spring-ThreadPool/)
-  [Asynchronous](blog/Spring-AsyncNonBlocking/)

#### 0. 들어가면서

  - 이번 포스트에서는 Spring에서 제공하는 비동기 처리 방법 및 종류에 대해 알아보겠습니다. 특히 @Async, ThreadPoolTaskExecutor, DefferedResult에 대해 정리하고 추후 포스트에서 Future과 Callable에 대해서 다루겠습니다.
  
  - ##### Asynchronous Technology in Java
    Servlet 3.0 이전에는 Worker Thread 없이 Servlet Thread만 존재했습니다. Servlet Thread가 Blocking 상태에 빠지게 됬을 때 많은 이슈들이 많이 생겼고 Servlet 3.0에서는 Servlet Thread와 Worker Thread가 분리되면서 비동기 처리를 지원하기 시작했습니다.

  - ##### Issue
    - ###### Procedure of Servlet 3.0
      - 1.Web Server의 NIO Connector에 의해 Request가 들어오고 Connection이 만들어진 후 Servlet Thread를 할당한다.
      - 2.오래걸리는 작업을 새로운 Worker Thread로 할당해서 처리한다.
      - 3.Servlet Thread는 Servlet Thread Pool에 반환된다.(해당 Servlet Thread로 다른 Request 처리 가능)
      - 4.Worker Thread는 AsyncContext를 통해서 Response를 추가하고 작업이 완료됨을 알린다.
      - 5.해당 Response를 Client에게 전달하고 Connection을 종료한다.
    - Servlet 3.0에서는 **Non-Blocking I/O**를 지원하지 않기 때문에 Worker Thread가 Blocking I/O 작업을 수행할 때 Blocking 상태에 놓인다.
      
  - ##### Solution
    - Servlet 3.1에서는 Non-Blocking I/O를 지원한다.
    - Spring 3.2에서는 Servlet 3.1에서 지원하는 **Non-Blocking**에 대한 기술을 활용할 수 있도록 Controller의 Handler가 다양한 Type의 객체를 반환할 수 있도록 지원한다. 해당 객체를 반환하게 되면 Spring이 비동기 작업을 실행해준다.

  
  - ##### Category
    - Java
      - ExecutorService
      - Future
      - FutureTask
    - Spring
      - **@Async**
      - **ThreadPoolTaskExecutor**
      - AsyncRestTemplate
      - Async Servlet(Return Type)
        - Return Type
          - Non-Blocking I/O
            - **DefferredResult(Single)**
            - ResponseBodyEmitter(Multi)
          - Blocking I/O
            - ListenableFuture
            - CompletableFuture
        - Callable
        - WebAsyncTask


<br />

---

#### 1. DeferredResult

  - **“지연된 결과”**를 의미하며 외부의 Event 혹은 Client Request에 의해서 지연되어 있는 HTTP 요청에 대한 응답을 나중에 써줄 수 있는 Spring 비동기 핵심 기술이다.
  - Return Type으로 DeferredResult가 아닌 Callable, WebAsyncTask, ListenableFuture, CompletableFuture를 사용할 경우 Worker Thread가 생성되고 작업을 실행한다. Worker Thread가 Blocking 작업을 수행할 때 해당 요청에 대한 응답을 받을 때까지 Blocking 상태에 놓이게 된다. 많은 수의 Request가 들어오고 Worker Thread의 수가 많아지면 리소스 낭비 및 CPU 사용량 증가를 초래한다.
  - **DeferredResult는 Worker Thread가 Blocking 상태에 놓이지 않고 Non-Blocking으로 처리할 수 있다.**
  - Worker Thread를 사용하지 않고 Servlet Thread도 즉시 반환한다.
    
  - ##### 1.1 FlowChart
    
    ![image info](/assets/img/DefferdResult.png)  
    <img src="/assets/img/DefferdResult.png" alt="" width="0" height="0">

    
    - **다른 Return Type과 다르게 1개의 Servlet Thread로 여러 Request를 처리하면서 Worker Thread를 계속해서 만들지 않는다.** 


  - ##### 1.2 Process

    - 1.Client의 Request가 들어옴
    - 2.Servlet Thread는 DeferredResult Queue에 Handler를 추가한다.
    - 3.Servlet Thread는 Servlet Thread Pool에 반환된다.
    - 5.외부에서 특정 Handler에 대한 Event가 발생한다.
    - 6.해당 Servlet Thread는 Queue에 저장된 객체에 값이 설정되며 즉시 결과를 받는 Response 처리 작업을 수행한다.

    
  - ##### 1.3 Reference

    - AsyncRestTemplate, WebClient는 1개의 Worker Thread로 Event Loop 형태로 처리한다.
    - DeferredResult는 다른 비동기 처리 기술과 결합되어 사용된다.
    - callback을 통해 처리 결과를 가공하거나 메소드 체이닝을 통해서 여러 Async-Nonblocking을 수행하는 작업에 어려움이 있다. 따라서 DeferredResult의 객체를 우선 반환하고 ListenableFuture, CompletableFuture의 메소드 체이닝을 통해 다양한 작업을 처리한 뒤에 callback을 통해서 .setResult() 할 수 있다.

  - ##### 1.4 Code

    ```java

    @SpringBootApplication
    @EnableAsync
    public class DeferredResultApplication {

      @RestController
      public static class DeferredResultController {
        Queue<DeferredResult<String>> results = new ConcurrentLinkedQueue<>();

        @PostMapping("/request")
        public DeferredResult<String> result() {
            DeferredResult<String> result = new DeferredResult<>();
            results.add(result);
            return result;
        }

        //외부에서 Event를 발생시키는 API
        @PostMapping("/response")
        public String ExternalEvent(String msg) {
            for (DeferredResult<String> result : results) {
                result.setResult(result);
                results.remove(result);
            }
            return "Complete";
        }
      }

      public static void main(String[] args) {
          SpringApplication.run(StudyApplication.class, args);
      }
    }
    ```

<br />

---

#### 2. @Async


  - Spring에서 제공하는 Thread Pool을 활용하는 비동기 메소드 지원 Annotation
  - Spring AOP에 의해 Proxy Pattern 기반으로 동작한다.
  - Spring은 해당 메소드을 별도의 실행 경로로 제출하기 위해 연결된 ThreadPool을 찾으려고 한다.
    - Default : SimpleAsyncTaskExecutor(지양)
  - SpringBoot에서의 사용
    - Application class에 @EnableAsync 추가
    - method에 @Async 추가


  - ##### 2.1 Process
    -1. @Async 메소드가 호출되면 Spring은 해당 호출을 가로채서 비동기 실행을 위한 Proxy 객체를 생성한다.
    -2. 해당 메소드는 TaskExecutor에 의해 ThreadPool 작업으로 등록한다.
    -3. 해당 메소드는 호출자와 별도의 Thread에서 작업이 진행되고 Blocking되지 않으며 즉시 리턴된다.


  - ##### 2.2 Code

    ```java
    @EnableAsync
    @SpringBootApplication
    public class SpringBootApplication {

    }
    public class AsyncClass {
        @Async
        public void asyncMethod() throws Exception {

        }
    }
    ```

  - ##### 2.3 Caution
    - 사용 시 Proxy로 만들 수 없는 경우
      - private method 적용이 불가하다.
      - self-invocation은 사용할 수 없다.
    - return type은 void 또는 CompletableFuture<>을 사용한다.

<br />

---

#### 3. ThreadPoolTaskExecutor

  - @Async를 사용한 메소드를 호출할 경우 ThreadPool이 선언되어 있지 않으면 SimpleAsyncTaskExecutor를 사용한다.
  - 해당 Executor는 호출마다 새로운 Thread를 만들어 사용하기 때문에 매우 비효율적이다.
  - SimpleAsyncTaskExecutor이 아닌 ThreadPoolTaskExecutor를 직접 만들어 사용하는 것을 지향한다.

  - ##### 3.1 Code

    ```java
    @Configuration
    @EnableAsync
    public class AsyncThreadConfiguration {
    	@Bean
    	public ThreadPoolTaskExecutor asyncThreadTaskExecutor() {
    		ThreadPoolTaskExecutor threadPoolTaskExecutor = new ThreadPoolTaskExecutor();
    		threadPoolTaskExecutor.setCorePoolSize(5);
    		threadPoolTaskExecutor.setMaxPoolSize(5);
    		threadPoolTaskExecutor.setThreadNamePrefix("MyThread");
        threadPoolTaskExecutor.initialize();

    		return threadPoolTaskExecutor;
    	}
    }
    ```

  - ##### 3.2 Options
    - corePoolSize : Thread Pool에 항상 살아있는 기본 Thread 수
    - maxPoolSize : Thread Pool이 확장할 수 있는 최대 Thread 수
    - queueCapacity : Thread Pool에서 사용할 최대 Queue 크기
    - threadNamePrefix : 생성된 Thread 이름의 접두사

  - ##### 3.3 Exception Handle
    - RejectedExecutionHandler를 통해 ThreadPoolTaskExecutor에서 Thread Pool내에서 더 이상  작업을 처리할 수 없을 때의 예외 처리를 할 수 있다.
    - Default : AbortPolicy

    - ##### Strategry
      - AbortPolicy : TaskRejectedException 발생 후 종료
      - CallerRunsPolicy : Thread Pool을 호출한 Thread에서 처리
      - DiscardPolicy : 해당 요청들을 무시
      - DiscardOldestPolicy : Queue에 있는 가장 오래된 요청을 삭제하고 새로운 요청을 받아들임

---

### Reference


- https://jongmin92.github.io/2019/03/31/Java/java-async-1/#Future
- https://hyokeun0419.tistory.com/105
- https://xxeol.tistory.com/44
- https://xxeol.tistory.com/44#%40EnableAsync%20%EC%A0%81%EC%9A%A9%20%EB%B0%A9%EB%B2%95-1
- https://ch4njun.tistory.com/267
- https://velog.io/@chanyoung1998/Spring-Async%EB%A1%9C-%EB%B9%84%EB%8F%99%EA%B8%B0-%EC%B2%98%EB%A6%AC
- https://xxeol.tistory.com/44#%40EnableAsync%20%EC%A0%81%EC%9A%A9%20%EB%B0%A9%EB%B2%95-1