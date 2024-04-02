---
title:  "Fork Join Pool"
excerpt: "[Spring] About Fork Join Pool "

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2024-03-10

---
#### 0. ForkJoinPool

> ForkJoinPool is a type of ExecutorService in Java that is designed to handle a large number of tasks in a highly efficient manner. It uses a divide-and-conquer approach to break down complex tasks into smaller, more manageable tasks. This approach helps to reduce the overhead associated with task execution and helps to optimize performance.

> ForkJoinPool은 자바에서 수많은 작업을 효율적으로 처리하도록 설계된 일종의 ExecutorService이다. Fork, Join 방식을 사용하여 복잡한 작업을 더 작고 관리하기 쉬운 작업으로 세분화합니다. 이 접근법은 작업 실행과 관련된 overhead를 줄이는 데 도움이 되며 성능을 최적화하는 데 도움이 된다.

<br />

---


### 1. ForkJoinPool


- #### 1.1 Fork, Join FlowChart

![image info](/assets/img/forkJoinPool.png)
<img src="/assets/img/forkJoinPool.png" alt="" width="0" height="0">

- ForkJoinPool은 **Fork**를 통해서 각각의 Thread에 task를 분담하여 처리하고 **Join**을 통해 task의 결과를 취합합니다.(Recursive)
- task -> subtask로 **fork**하여 분리되고 subtask는 **parallel** 또는 **concurrent**하게 실행됩니다.
- task가 subtask로 분리된 후 task는 subtask가 종료할 때까지 **wait**합니다.
- subtask가 종료되고 task는 결과를 **join**합니다.



<br />

- #### 1.2 Fork, Join Algorithm

![image info](/assets/img/forkJoinPool2.png)
<img src="/assets/img/forkJoinPool2.png" alt="" width="0" height="0">

- Procedure
  1. Task를 submit하면 하나의 queue(inbound queue)에 task가 누적됩니다.
  2. Thread A,B 는 task를 가져와서 각자의 task queue(Dequeue)에 가져와서 처리합니다.
  3. Thread B queue에는 task가 비어있는 상태이므로 Thread A queue의 task를 가져옵니다.
  4. Thread의 task queue는 Dequeue이므로 task가 없는 다른 Thread에서 task를 가져갈 수 있습니다.

- Fork, Join은 최대한 task가 비어 있는 상태를 최소화하기 위한 방식입니다.


<br />

---
### 2. ForkJoinPool Interface

- ForkJoinPool에서는 2가지 Interface를 지원합니다.
  - RecursiveTask : return이 있는 task
  - RecursiveAction : return이 없는 task

```java
ForkJoinPool pool = new ForkJoinPool();
pool.submit(task);
//Future 개체 반환(ForkJoinPool의 작업 상태 확인, 작업 결과 확인)
```

<br />

- #### 2.1 RecursiveTask


```java
//RecursiveTask.java
package java.util.concurrent;

public abstract class RecursiveTask<V> extends ForkJoinTask<V> {
    private static final long serialVersionUID = 5232453952276485270L;

    V result;

    protected abstract V compute();

    public final V getRawResult() {
        return result;
    }

    protected final void setRawResult(V value) {
        result = value;
    }

    protected final boolean exec() {
        result = compute();
        return true;
    }

}
```

```java
public class MyRecursiveTask extends RecursiveTask<Long> {

    private long workLoad = 0;

    public MyRecursiveTask(long workLoad) {
        this.workLoad = workLoad;
    }

    protected Long compute() {
        String threadName = Thread.currentThread().getName();

        //if work is above threshold, break tasks up into smaller tasks
        if (this.workLoad > 16) {
            System.out.println("[" + LocalTime.now() + "][" + threadName + "]"
                    + " Splitting workLoad : " + this.workLoad);
            sleep(1000);
            List<MyRecursiveTask> subtasks =
                    new ArrayList<MyRecursiveTask>();
            subtasks.addAll(createSubtasks());

            for (MyRecursiveTask subtask : subtasks) {
                subtask.fork();
            }

            long result = 0;
            for (MyRecursiveTask subtask : subtasks) {
                result += subtask.join();
                System.out.println("[" + LocalTime.now() + "][" + threadName + "]"
                        + "Received result from subtask");
            }
            return result;

        } else {
            sleep(1000);
            System.out.println("[" + LocalTime.now() + "]["
                    + " Doing workLoad myself: " + this.workLoad);
            return workLoad * 3;
        }
    }

    private List<MyRecursiveTask> createSubtasks() {
        List<MyRecursiveTask> subtasks =
                new ArrayList<MyRecursiveTask>();

        MyRecursiveTask subtask1 = new MyRecursiveTask(this.workLoad / 2);
        MyRecursiveTask subtask2 = new MyRecursiveTask(this.workLoad / 2);

        subtasks.add(subtask1);
        subtasks.add(subtask2);

        return subtasks;
    }

    private void sleep(int millis) {
        try {
            Thread.sleep(millis);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
```java
public static void main(String[] args) {
    ForkJoinPool forkJoinPool = new ForkJoinPool(4);

    MyRecursiveTask myRecursiveTask = new MyRecursiveTask(128);
    long mergedResult = forkJoinPool.invoke(myRecursiveTask);
    System.out.println("mergedResult = " + mergedResult);

    try {
        forkJoinPool.awaitTermination(5, TimeUnit.SECONDS);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```


<br />

---

- #### 2.2 RecursiveAction

```java
//RecursiveAction.java
package java.util.concurrent;

public abstract class RecursiveAction extends ForkJoinTask<Void> {
    private static final long serialVersionUID = 5232453952276485070L;

    protected abstract void compute();


    public final Void getRawResult() { return null; }


    protected final void setRawResult(Void mustBeNull) { }

    protected final boolean exec() {
        compute();
        return true;
    }

}
```

```java
public class MyRecursiveAction extends RecursiveAction {

    private long workLoad = 0;

    public MyRecursiveAction(long workLoad) {
        this.workLoad = workLoad;
    }

    @Override
    protected void compute() {
        String threadName = Thread.currentThread().getName();

        //if work is above threshold, break tasks up into smaller tasks
        if(this.workLoad > 16) {
            System.out.println("[" + LocalTime.now() + "][" + threadName + "]"
                    + " Splitting workLoad : " + this.workLoad);
            sleep(1000);
            List<MyRecursiveAction> subtasks = new ArrayList<MyRecursiveAction>();

            subtasks.addAll(createSubtasks());

            for(RecursiveAction subtask : subtasks){
                subtask.fork();
            }

        } else {
            System.out.println("[" + LocalTime.now() + "][" + threadName + "]"
                    + " Doing workLoad myself: " + this.workLoad);
        }
    }

    private List<MyRecursiveAction> createSubtasks() {
        List<MyRecursiveAction> subtasks =
                new ArrayList<MyRecursiveAction>();

        MyRecursiveAction subtask1 = new MyRecursiveAction(this.workLoad / 2);
        MyRecursiveAction subtask2 = new MyRecursiveAction(this.workLoad / 2);

        subtasks.add(subtask1);
        subtasks.add(subtask2);

        return subtasks;
    }

    private void sleep(int millis) {
        try {
            Thread.sleep(millis);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
```java
public static void main(String[] args) {
    ForkJoinPool forkJoinPool = new ForkJoinPool(4);

    MyRecursiveAction myRecursiveAction = new MyRecursiveAction(128);
    forkJoinPool.invoke(myRecursiveAction)

    try {
        forkJoinPool.awaitTermination(5, TimeUnit.SECONDS);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```




<br />

---


### Refernce 
- https://junghyungil.tistory.com/103
- https://phantasmicmeans.tistory.com/entry/ForkAndJoinPool-%EC%9D%B4%EB%9E%80
- https://hamait.tistory.com/612
- https://willbfine.tistory.com/469
- https://wiki.yowu.dev/ko/Knowledge-base/Java/leveraging-java-s-java-util-concurrent-forkjoinpool-for-recursive-task-processing