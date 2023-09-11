---
title:  "Spring Container"
excerpt: "[Spring] Spring Container"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2023-09-11

---


#### 1. Spring Container(Spring Core)

> The Spring container is at the core of the Spring Framework. The container will create the objects, wire them together, configure them, and manage their complete life cycle from creation till destruction. The Spring container uses DI to manage the components that make up an application. These objects are called Spring Beans.

> Spring Conatiner는 Spring Framework의 핵심에 있다. Spring Container는 객체를 만들고, 연결하고, 구성하고, 만들어지는 것부터 파괴될 때까지 완전한 수명 주기를 관리할 것이다. Spring Container는 DI를 사용하여 응용 프로그램을 구성하는 구성 요소를 관리한다. 이 객체들을 Spring Beans라고 부른다.


![image info](/assets/img/springFramework.png)
<img src="/assets/img/springFramework.png" alt="" width="0" height="0">

- Spring Core가 Spring Container를 의미합니다
  - Spring Container는 Spring Framework의 핵심 Component
- Spring Container는 자바 객체의 생명 주기를 관리하며, 생성된 자바 객체들에게 추가적인 기능을 제공하는 역할을 합니다
- 위의 자바 객체는 빈(Bean)이라고 합니다
- Singleton Container
  - 객체의 instance를 Singleton으로 관리합니다
  - Bean들의 주소는 동일합니다
- Spring Container에는 IoC와 DI의 원리가 적용됩니다
- 개발자 대신 빈의 생명 주기를 관리 해줍니다


```java
@Configuration
public class AppConfig{
    @Bean
    public BeanExampleService beanExampleService(){
        return new BeanExampleServiceImpl(examplePolicy());
    }
    @Bean
    public SimpleExamplePolicy examplePolicy(){
        return new SimplerExamplePolicy();
    }
}
```

```java
ApplicationContext springContainerExample = new AnnotationConfigApplicationContext(AppConfig.class);

//빈 이름으로 조회
BeanExampleService beanExampleService = springContainerExample.getBean("beanExampleService", BeanExampleService.class);
```

<br />

---


### 2. Spring Container Type

![image info](/assets/img/springContainer.png)
<img src="/assets/img/springContainer.png" alt="" width="0" height="0">


- Spring Container는 2가지 Interface로 구현 되어 있습니다
  - BeanFactory Interface
  - ApplicationContext Interface

##### 2.1 BeanFactory

- Spring Container의 최상위 Interface입니다
- Bean의 등록, 생성, 조회, 소멸 등의 생명 주기를 관리합니다
- @Bean을 통해서 Bean을 등록합니다
- getBean()을 통해서 Bean instance

<br />

##### 2.2 ApplicationContext

- BeanFactory를 구현하고 있는 Interface입니다(ApplicationContext >>> Bean)
  - BeanFactory의 확장 버전
  - BeanFactory의 모든 기능을 포함하고 추가적인 기능을 제공합니다.
    - Bean 관리 + 편리한 부가기능
- 추가적인 기능
  - MessageSource : 국제화 기능(언어)
  - EnvironmentCapable : 환경 변수(local, dev, prod)
  - ApplicationEventPublisher : Event 생성
  - ResourceLoader : File, ClassPath, External Resource 조회


<br />


---

### Refernce 
- https://www.tutorialspoint.com/spring/spring_ioc_containers.htm
- https://steady-coding.tistory.com/459
- https://velog.io/@alivejuicy/Spring-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88
- https://ittrue.tistory.com/220