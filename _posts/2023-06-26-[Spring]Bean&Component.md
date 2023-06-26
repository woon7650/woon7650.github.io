---
title:  "[Spring]@Bean & @Component"
excerpt: "About @Bean & @Component"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2023-06-26
---


## 0. 들어가면서


##### @Bean & @Component

- @Bean과 @Component는 Spring Container의 객체를 생성하는 기능을 함
- 비슷한 기능을 하는 @Bena과 @Component에 대하여 차이점을 알아보고자 함

<br />

---


# 1. @Bean

##### 1.1 @Bean

- 개발자가 제어하지 못하는 <mark style="background-color:#cccccc">외부 클래스, 라이브러리</mark>를 Spring Container의 Bean으로 등록하고 싶은 경우 사용
- Spring Container에서 자동으로 감지하는 것이 아닌 <mark style="background-color:#cccccc">명시적으로 정의</mark>해야 함
- @Bean 어노테이션이 적용된 Method는 Spring Container에 의해 호출되며 반환되는 객체는 Bean으로 등록됩니다

<br />

##### 1.2 Target of @Bean


```java
package org.springframework.context.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import org.springframework.beans.factory.annotation.Autowire;
import org.springframework.core.annotation.AliasFor;

@Target({ElementType.METHOD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Bean {
    @AliasFor("name")
    String[] value() default {};

    @AliasFor("value")
    String[] name() default {};

    /** @deprecated */
    @Deprecated
    Autowire autowire() default Autowire.NO;

    boolean autowireCandidate() default true;

    String initMethod() default "";

    String destroyMethod() default "(inferred)";
}

```

- <mark style="background-color:#cccccc">@Bean은 Class에 사용 불가(Method, Annotation type에 사용 가능)</mark>

<br />

##### 1.3 Example of @Bean

```java
import org.springframework.boot.autoconfigure.jdbc.DataSourceProperties;

@Configuration
public class DatasourceConfig {

    @Bean
    public DataSourceProperties dataSourceProperties() {
        return new DataSourceProperties();
    }
}
```

- @Bean 사용 시 <mark style="background-color:#cccccc">@Configuration</mark>을 꼭 사용해야 함(사용하지 않으면 의존성 주입에 활용 불가)
- @Configuration 안에는 Spring Container에 들어간 Bean이 필요, 객체를 생성하는 <mark style="background-color:#cccccc">method</mark>에 @Bean 부여


<br />
---

# 2. @Component

##### 2.1 @Component

- 개발자가 제어 가능한 <mark style="background-color:#cccccc">Class</mark>를 Spring Container의 Bean으로 등록하고 싶은 경우 사용
- Spring Container에서 <mark style="background-color:#cccccc">자동으로 감지</mark>
- @Component가 적용된 Class는 Spring Container에 의해 객체로 인스턴스화되어 관리됨


<br />

##### 2.2 Target of @Component

```java
package org.springframework.context.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import org.springframework.core.annotation.AliasFor;
import org.springframework.stereotype.Component;

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Configuration {
    @AliasFor(
        annotation = Component.class
    )
    String value() default "";

    boolean proxyBeanMethods() default true;
}

```

- <mark style="background-color:#cccccc">@Component는 개발자가 제어할 수 있는 Class 위에 사용 가능</mark>

<br />

##### 2.3 Related Annotation

> 객체 성격에 따라 사용되는 Annotation이 다름(해당 Annotation들을 통해 Bean 생성)

- <mark style="background-color:#cccccc">@Configuation</mark>
- <mark style="background-color:#cccccc">@Controller</mark>
- <mark style="background-color:#cccccc">@RestController</mark>
- <mark style="background-color:#cccccc">@Service</mark>
- <mark style="background-color:#cccccc">@Repository</mark>

<br />

##### 2.4 Example of @Component

```java
@Component
public class Example {

    private TestCode testCode;

    @Autowired
    public Example(TestCode testCode) {
        this.testCode = testCode;
    }
}
```

- Example는 내가 직접 만든 Class이므로 @Component 사용 가능  
- @Component를 통해서 Example Class는 Spring Container의 빈으로 등록됨
- <mark style="background-color:#cccccc">@Autowired</mark>를 통해서 Spring Container가 ' TestCode '타입의 Bean을 찾아 자동으로 주입
- ' Example ' Class에 ' testCode '를 사용할 수 있게 됨



---
