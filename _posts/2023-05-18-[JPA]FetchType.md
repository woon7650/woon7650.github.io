---
title:  "[JPA] FetchType"
excerpt: "About FetchType"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2023-05-18
---


## FetchType

---
### FetchType이란 ?

<br />

- 하나의 Entity를 조회할 때 연관 관계에 있는 객체들을 어떻게 가져올 것인지 지정하는 JPA의 option
- FetchType.LAZY 와 FetchType.EAGER 두가지 유형의 Fetch 방식을 제공
- 데이터의 일관성을 유지하고 연관된 Entity간의 작업을 간편하게 처리하기 위해 사용함
- @ManyToOne, @OneToOne, @OneToMany, @ManyToMany 등 에서 사용함

<br />

---
### FetchType의 종류

<br />

*LAZY(지연로딩)*

- FetchType.LAZY
- 연관된 Entity가 실제로 필요한 시점에만 data를 로딩함(getter로 접근할 때 가져옴)
- 연관된 Entity에 처음 접근하는 순간에 DB에서 쿼리를 실행하여 로딩함

<br />

*EAGER*

- FetchType.EAGER
- 부모 Entity를 조회할 때 연관된 모든 자식 Entity도 함께 로딩함
- 모든 데화욜에 그럼 좀일찍 만나서 10시쯤 파하던지이터를 한 번에 조회함


<br />

> Example Code ( FetchType.LAZY & FetchType.EAGER )

<br />

**Example Entity는 Test Entity를 Eager로 조회하고 Test Entity는 Example Entity를 Lazy로 조회함**
```java
@Entity
public class Example {

  @Id
  private Long id;

  @ManyToOne(fetch = FetchType.EAGER)
  private Test test;

}

@Entity
public class Test {

  @Id
  private Long id;

  @OneToMany(mappedBy = "example", fetch = FetchType.LAZY)
  private List<Example> example;

}
```

<br />

---
### FetchType 사용 시 주의할 점

<br />

*FetchType.LAZY*

- 영속성 컨텍스트의 범위와 접근 시점에 주의해야 함
- LazyInitializationException가 발생하지 않도록 해야 함

<br />

*FetchType.EAGER*

- 모든 연관 Entity를 즉시 로딩하기 때문에 성능 저하의 가능성이 있을 수 있음
- 필요한 경우에만 FetchType.EAGER를 사용하는 것을 권장함

---
### One-to-Many & Many-to-One

<br />

*One-to-Many*

- 주인 Entity는 FetchType.LAZY로 조회해야 함
- FetchType.LAZY로 조회 안할 때 N+1 조회 문제가 발생할 수 있음

*Many-to-One*

- 주인 Entity는 FetchType.EAGER로 조회해야 함
- FetchType.EAGER로 조회 안할 때 연관된 Entity가 조회되지 않음


*One-to-One*

- 주인 Entity는 FetchType.LAZY, FetchType.EAGER로 조회할 수 있음

<br />

> N+1 문제
> - 연관된 Entity를 한 번에 하나씩 조회하는 문제
> - 1개의 Test에 각각 5개의 Example이 존재할 때 5개의 Test를 조회하고 1개의 Test에 대하여 각각 5번씩 Query를 실행해서 Test와 Example을 모두 조회함


<br />

---
