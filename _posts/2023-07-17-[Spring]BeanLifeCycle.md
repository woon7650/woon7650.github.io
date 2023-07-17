---
title:  "Bean LifeCycle"
excerpt: "[Spring]Spring Basics Part 1"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2023-07-17
---


## 0. 들어가면서


##### Bean LifeCycle

- 해당 객체가 생성되고 소멸될까지의 과정을 나타냄(when, how)
- Spring은 3가지 방법으로 Bean LifeCycle Callback을 관리

<br />

---


# 1. Bean LifeCycle

### 1.1 Bean LifeCycle Flow


![image info](/assets/img/beanLifeCycle.png)
<img src="/assets/img/beanLifeCycle.png" alt="" width="0" height="0">


1. Create Spring Container
2. Create Spring Bean
3. Dependencies Injected
4. Bean Initialize  : After 2,3 step
5. Use
6. Callback Before Destroy : Before Bean Destroyed
7. Spring Off

<br />

---

### 2. Callback

<br />

- Spring Framework는 Bean LifeCycle 제어를 위해 다음과 같은 방법을 제공 

> 1. Interfaces( InitializingBean & DisposableBean )
> 2. Setting Method( initialize() &  close() )
> 3. Annotation( @PostConstruct & @PreDestroy )

<br />

##### 2.1 InitializingBean & DisposableBean

<br />

- <mark style="background-color:#cccccc">InitializingBean</mark> : Bean에 필요한 속성들이 Container에 설정된 후 Bean Initialize Callback

- <mark style="background-color:#cccccc">DisposableBean</mark> : Spring Container가 Bean을 소멸하기 전 Bean Destroy Callback


```java
public class Example implements InitializingBean, DisposableBean {
    public Example()  {
        System.out.println("Constructor Callback");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("Initialization");
        //Bean Initialization Code
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("Destruction");
        //Bean Destruction Code
    }
}
```

- Interface를 이용한 Initialize, Destroy 방법(Spring 초창기에 사용하던 방법)

<br />

##### 2.2 initialize() & destroy()

<br />

- @Bean에 <mark style="background-color:#cccccc">initMethod, destroyMethod</mark> 속성을 이용하여 Initialize, Destroy

- @Bean(initMethod = "initialize", destroyMethod = "close") 


```java
public class Example{

    public void initialize() throws Exception{
        System.out.println("Initialization");
        //Bean Initialization Code
    }
    public void close() throws Exception{
        System.out.println("Destruction");
        //Bean Destruction Code
    }
}
```

```java
@Configuration
class LifeCycleConfigi{
    @Bean(initMethod = "initialize", destroyMethod = "close")
    public Example example(){
    
    }
}
```



<br />

##### 2.3 @PostConstruct & @PreDestroy

<br />

  - <mark style="background-color:#cccccc">@PostConstruct</mark> : Bean이 생성 후 Instance가 요청 객체에 반환되기 직전에 Callback

- <mark style="background-color:#cccccc">@PreDestroy</mark> : Bean Container 내부에서 Bean Destroy 직전에 Callback

```java
import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

public class Example{
	@PostConstruct
	public void customInit(){
		System.out.println("Customized Initialize Method Callback");
	}

	@PreDestroy
	public void customDestroy(){
		System.out.println("Customized Destroy Method Callback");
	}
}
```

- Annotation을 통해서 Callback 함수들을 Cuustomize(최신 스프링에서 권장하는 방식)

<br />

---

### 3. Comparation

<br />

||Interface|Config Method|Annotation|
|---|---|---|---|
|Usage In External Library|X|O|X|
|Spring Code Dependency|O|X|X|

- 코드 수정이 불가능한 외부 라이브러리에는 설정 정보에 초기화, 종료 메소드 지정 방식 방식을 사용하는 것이 좋음
- initialize() &  destroy()

<br />

---