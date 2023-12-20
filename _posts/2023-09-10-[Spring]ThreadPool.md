---
title:  "ThreadPool & ExecutorService"
excerpt: "[Spring] About Thread Pool and ExecutorService"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2023-09-10

---
### 1. Thread Pool

#### 1.1 Thread Pool

> Thread Pool은 동시에 여러 작업을 효율적으로 실행 및 관리하기 위해 서버에서 만드는 Thread의 모음입니다. Thread Pool을 이용하여 각 작업에 대해 새로운 Thread를 생성하는 대신 Thread Pool에 이미 생성 되어 있는 Thread를 재사용합니다. 


![image info](/assets/img/ThreadPool.png)
<img src="/assets/img/ThreadPool.png" alt="" width="0" height="0">

- Thread Pool은 여러 개의 Thread를 유지, 관리함
- Thread Pool내에 있는 Thread들은 작업이 할당되기를 기다림
- 장점
  - Thread를 생성/수거하는데 드는 비용 감소
    - 이전의 Thread를 재사용할 수 있음
  - 작업 딜레이가 발생하지 않음
    - 작업 요청시 이미 Thread가 대기 중인 상태
- 단점
  - Memory 낭비가 발생할 수 있음
    - Thread를 너무 많이 생성해 두고 사용하지 않을 경우
  - 최적의 Thread Pool Size를 설정하기 어려움

<br />

#### 1.2 Structure in Java

![image info](/assets/img/ThreadPoolJava.png)
<img src="/assets/img/ThreadPoolJava.png" alt="" width="0" height="0">


- Executors Class를 통해 생성한 Thread Pool은 ExecutorService interface를 구현한 instance
- Blocking Queue에서 작업을 꺼내 Thread에 전달
- Thread Pool instance에는 BlockingQueue, Worker Thread, HashSet이 존재
  - Blocking Queue : 처리할 작업들이 들어있는 곳
  - HashSet : Worker Thread들이 담긴 곳

<br />

---

### 2. Executors, ExectorService

> Java에서는 Thread Pool을 사용할 수 있도록 <mark style="background-color:#cccccc">java.util.concurrent.Executors</mark> Class와 <mark style="background-color:#cccccc">java.util.concurrent.ExecutorService</mark> Interface를 지원합니다. Executors에서 제공하는 static method들을 통해 ExecutorService 구현 instance를 만들어서 사용할 수 있습니다.

- JDK 1.5부터 java.util.concurrent pacakge를 제공합니다.
- java.util.concurrent
  - Executors class
    - static factory method를 제공
      - newFixedThreadPool
      - newCachedThreadPool
      - newScheduledThreadPool
      - newSingleThreadExecutor
  - ExecutorService interface
    - Thread Pool이 제공하는 기능들을 정의
    - execute()
      - Runnable을 작업 Queue에 저장
      - return 없음
      - ExecutorException 발생할 경우 Thread 종료 및 Thread 제거, 새로운 Thread 생성
    - submit()
      - Runnable or Callable 작업 Queue에 저장
      - return 있음(Future<?>, Future<V>)
      - ExecutorException 발생할 경우 Thread는 종료되지 않고 다음 작업을 위해 재사용
    - shutdown()
      - 더 이상 작업을 받지 않고 처리중인 작업 모두 완료 후 Thread Pool을 종료
    - shutdownNow()
      - 모든 Thread가 동시에 종료되는 것을 보장하지 않고 즉시 종료 시도
      - 실행되지 않은 작업을 반환
  - ThreadPoolExecutor class
    - ExecutorService Interface의 구현체 class
  - CompletionService Interface
    - poll(), take()를 이용하여 처리완료된 작업만 통보 가능


<br />

#### 2.1 ThreadPoolExecutor

```java
public class ThreadPoolExecutor extends AbstractExecutorService {
  public ThreadPoolExecutor(int corePoolSize,
                            int maximumPoolSize,
                            long keepAliveTime,
                            TimeUnit unit,
                            BlockingQueue<Runnable> workQueue,
                            ThreadFactory threadFactory,
                            RejectedExecutionHandler handler) {
      if (corePoolSize < 0 ||
          maximumPoolSize <= 0 ||
          maximumPoolSize < corePoolSize ||
          keepAliveTime < 0)
          throw new IllegalArgumentException();
      if (workQueue == null || threadFactory == null || handler == null)
          throw new NullPointerException();
      this.corePoolSize = corePoolSize;
      this.maximumPoolSize = maximumPoolSize;
      this.workQueue = workQueue;
      this.keepAliveTime = unit.toNanos(keepAliveTime);
      this.threadFactory = threadFactory;
      this.handler = handler;
  }
}
```

- corePoolSize : Thread Pool 내에서 반드시 존재해야하는 Thread 개수
- maximumPoolSize : Thread Pool에 저장할 수 있는 최대 Thread 개수
- keepAliveTime : Thread 수가 Core 수보다 많을 때 쉬는 Thread가 종료되기 전에 새 작업을 기다리는 최대 시간
- uint : keepAliveTime의 시간 단위
- workQueue : 처리되야 하는 작업들이 저장되있는 Queue
- threadFactory : Thread 생성 시 사용하는 Factory instance
- handler : Queue 용량이 초과되어 작업 실행이 block될 때 사용되는 handler instance

<br />

#### 2.3 ExecutorService

```java
public interface ExecutorService extends Executor {

    void shutdown();

    List<Runnable> shutdownNow();

    boolean isShutdown();

    boolean isTerminated();

    boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException;

    <T> Future<T> submit(Callable<T> task);

    <T> Future<T> submit(Runnable task, T result);

    Future<?> submit(Runnable task);

    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks) throws InterruptedException;

    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,long timeout, TimeUnit unit) throws InterruptedException;

    <T> T invokeAny(Collection<? extends Callable<T>> tasks) throws InterruptedException, ExecutionException;

    <T> T invokeAny(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;
}
```

<br />

#### 2.2 Executors

##### 2.2.1 Cached Thread Pool

```java
public class Executors {
  public static ExecutorService newCachedThreadPool() {
      return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                    60L, TimeUnit.SECONDS,
                                    new SynchronousQueue<Runnable>());
  }
}
```

- Thread를 Caching하는 ThreadPoolExecutor instance
- Thread Pool에서 반드시 유지되어야 하는 코어 Thread 개수 0
- 60s 동안 작업이 없으면 Thread Pool에서 제거
- SynchronousQueue 인스턴스에 작업을 담음
- 필요할 때 필요한 만큼 Thread를 생성
  - Thread를 개수 제한 없이 무한정 생성해서 폭발적으로 증가할 수 있음

```java
//Test code
void submitTasks(int taskAmount, ExecutorService executorService, Runnable runnable) {
    for (int index = 0; index < taskAmount; index++) {
        executorService.submit(runnable);
    }
}

void threadExampleCode() {
    int taskAmount = 20;
    ExecutorService executorService = Executors.newCachedThreadPool();
    submitTasks(taskAmount, executorService, () -> {
        log.info("Task running");
    });
}
```

- 작업이 들어올 때 모두 다른 Thread 사용
- 작업을 모두 전달한 시점의 Thread Pool Size는 20


<br />


##### 2.2.2 Fixed Thread Pool

```java
public class Executors {
  public static ExecutorService newFixedThreadPool(int nThreads) {
      return new ThreadPoolExecutor(nThreads, nThreads,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>());
  }
}
```

- 고정된 Thread 개수를 생성하는 ThreadPoolExecutor instance
- Thread Pool에서 반드시 유지되어야 하는 코어 Thread 개수 nThreads 값
- 0s 동안 작업이 없으면 Thread 종료
- 작업이 없을 때 작업을 얻을 때까지 Thread들이 대기
- LinkedBlockingQueue 인스턴스에 작업을 담음
- CPU 코어 수를 기준으로 생성하면 좋음


```java
//Test code
void submitTasks(int taskAmount, ExecutorService executorService, Runnable runnable) {
    for (int index = 0; index < taskAmount; index++) {
        executorService.submit(runnable);
    }
}

void threadExampleCode() {
    int taskAmount = 20;
    ExecutorService executorService = Executors.newFixedThreadPool(10);
    submitTasks(taskAmount, executorService, () -> {
        log.info("Task running");
    });
}
```

- 20개의 작업을 10개의 Thread가 나눠서 수행
- 작업을 모두 전달한 시점의 Thread Pool Size는 10

<br />


##### 2.2.3 Scheduled Thread Pool

###### 2.2.3.1 if CORE_FULL_SIZE > 0

```java
public class Executors {
  public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
      return new ScheduledThreadPoolExecutor(corePoolSize);
  }
}
```
- 일정 시간 뒤에 실행되는 작업, 주기적으로 실행되는 작업에 사용하는 ScheduledThreadPoolExecutor instance
- Thread Pool에서 반드시 유지되어야 하는 코어 Thread 개수 corePoolSize 값
- 10s 동안 작업이 없으면 Thread 종료
- 작업이 없을 때 작업을 얻을 때까지 Thread들이 대기
- DelayedWorkQueue 인스턴스에 작업을 담음
- CPU 코어 수를 기준으로 생성하면 좋음


```java
//Test code
void submitTasks(int taskAmount, ExecutorService executorService, Runnable runnable) {
    for (int index = 0; index < taskAmount; index++) {
        executorService.submit(runnable);
    }
}

void threadExampleCode() {
    int taskAmount = 20;

    ScheduledExecutorService executorService = Executors.newScheduledThreadPool(10);
    executorService.schedule(() -> {
        log.info("Scheduled Task");
    }, 2000, TimeUnit.MILLISECONDS);
    submitTasks(taskCount, executorService, () -> {
        log.info("Task running");
    });
}
```

- 20개의 작업을 10개의 Thread가 나눠서 수행
- 작업을 모두 전달한 시점의 Thread Pool Size는 10
- 2s 뒤에 지정된 작업 수행

<br />

###### 2.2.3.1 if CORE_FULL_SIZE = 0




<br />

##### 2.2.4 Single Thread Pool

```java
public class Executors {
  public static ExecutorService newSingleThreadExecutor() {
      return new FinalizableDelegatedExecutorService
          (new ThreadPoolExecutor(1, 1,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>()));
  }
}
```
- Thread가 1개인 ThreadPool
- 실패할 경우 새로운 Thread 생성하지 않음

<br />

---

### Reference

- https://en.wikipedia.org/wiki/Thread_pool
- https://www.baeldung.com/thread-pool-java-and-guava
- https://cheershennah.tistory.com/170
- https://change-words.tistory.com/entry/Tread-Pool#google_vignette
- https://velog.io/@backtony/Java-ExecutorService%EC%99%80-ForkJoinPool
- https://medium.com/@itsmynameangad/java-build-your-own-custom-executorservice-threadpool-213e46159326
- https://jeong-pro.tistory.com/188