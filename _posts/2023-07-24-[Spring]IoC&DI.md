---
title:  "Ioc & DI"
excerpt: "[Spring] About Spring Basics Part 2"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2023-07-24
---


#### 0. 들어가면서

- Spring의 기본적인 개념에 대해서 한 번 정리하고자 함

#### Ioc(Inversion of Control) & DI(Dependency Injection)

- DI는 Ioc를 구현하기 위해 사용하는 디자인 패턴 중 하나로 객체의 의존관계를 외부에서 주입시키는 패턴

- DI를 통해 Ioc를 구현함으로써 아래와 같은 이점들을 취함

  - 의존성이 줄어듬
  - 단위 테스트가 쉬워짐
  - 가독성이 높아짐
  - 재사용성이 높아짐

---


### 1. Ioc(Inversion of Control)


##### 1.1 Ioc(Inversion of Control)?

- 개발자가 제어하지 못하는 <mark style="background-color:#cccccc">외부 클래스</mark>, <mark style="background-color:#cccccc">라이브러리</mark>를 Spring Container의 Bean으로 등록하고 싶은 경우 사용
- 개발자가 객체를 생성하지 않고 객체의 생명주기를 관리하는 Spring에 위임
- 객체의 생명주기를 Spring에서 관리하기 때문에 개발자는 Business Logic 작성에 집중할 수 있음

<br />

##### 1.2 Example


```java
public class ExampleServiceImpl implements ExampleService{

    private final SelectCode selectCode;
    public ExampleServiceImpl(SelectCode selectCode){
        this.selectCode = selectCode;
    }


}
```

```java
public class Config{
    public ExampleService exampleService(){
        return new ExampleServiceImpl(selectCode());
    }
    public Code code(){
        return new ExampleCode1();
        //return new ExampleCode2();
    }
}
```

- Example Code를 무엇을 사용할지는 구현 객체(ExampleServiceImpl)가 결정하는 것이 아닌 Config에서 객체를 생성하고 제어함
- <mark style="background-color:#cccccc">제어 흐름</mark>을 직접 제어하는 것이 아닌 외부에서 관리함

<br />

---

## 2. DI(Dependency Injection)

##### 2.1 DI(Dependency Injection)?

> 의존 대상 B가 변하면, 그것이 A에 영향을 미친다.
> (from 토비의 스프링)

- Spring Container에서 관리할 객체를 지정해주고, Spring Container에서 생성된 객체를 받아서 사용하는 방식
- 개발자에게는 DI에 대한 제어권이 없음
  



<br />

##### 2.2 Constructor Injection

```java
@RestController
public class ExampleController {
    private final ExampleService exampleService;
    
    public ExampleController(ExampleService exampleService){
        this.exampleService = exampleService;
    }
}
```

- Constructor는 객체 생성시 1번만 호출되고 다시 호출되는 일이 없기 때문에 final 생성이 가능
- 순환 참조 오류 방지(생성자 주입 사용을 지향)
- Lombok을 통해 간결하게 작성 가능

<br />

##### 2.3 Setter Injection

```java
@RestController
public class ExampleController {
    private ExampleService exampleService;
    
    @Autowired
    public void setExampleController(ExampleService exampleService){
        this.exampleService = exampleService;
    }
}
```

- setter에 @Autowired를 같이 사용

<br />

##### 2.4 Interface Injection

```java
public interface BInjection {
    void inject(B b);
}

public A implements BInjection {
    private B b;
    
    @Override
    public void inject(B b) {
        this.b = b;
    }
}
```

- Interface의 Override를 통한 의존성 주입


<br />

##### 2.5 Field Injection

```java
@RestController
public interface ExampleController {

    @Autowired
    private ExampleService exampleService;

}
```
- Field에 @Autowired를 선언하여 의존성 주입을함
- 쉽게 의존성 주입이 가능하지만 단일 책임 원칙을 위반할 수 있음(@Autowired 사용은 지양)


---