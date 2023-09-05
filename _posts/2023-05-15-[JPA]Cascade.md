---
title:  "Cascade"
excerpt: "[JPA] About Cascade(영속성 전이)"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2023-05-15
---

## Cascade란 ?

<br />

- Entity 간 연관 관계에서 발생하는 작업을 관리하는 JPA의 option
- Cascade option에 따라서 부모 Entity에 대한 특정 작업이 자동으로 연관된 자식 Entity에 영향을 줌
- 데이터의 일관성을 유지하고 연관된 Entity간의 작업을 간편하게 처리하기 위해 사용함

<br />

---
## Cascade의 종류

<br />

> ALL(모든 작업 전파)

- cascade = CascadeType.ALL
- 부모 Entity에 대한 저장, 업데이트, 삭제 작업이 자식 Entity에 동일하게 전파됨

> PERSIST(저장 작업 전파)

- cascade = CascadeType.PERSIST
- 부모 Entity가 저장 시  자식 Entity도 자동으로 저장됨

> REMOVE(삭제 작업 전파)

- cascade = CascadeType.REMOVE
- 부모 Entity 삭제 시 자식 Entity도 자동으로 삭제됨

> MERGE(병합 작업 전파)

- cascade = CascadeType.MERGE
- 부모 Entity 병합 시  자식 Entity도 자동으로 병합됨

> REFRESH(새로고침 작업 전파)

- cascade = CascadeType.REFRESH
- 부모 Entity 새로 고침시  자식 Entity도 자동으로 새로 고침됨

> DETACH(분리 작업 전파)

- cascade = CascadeType.DETACH
- 부모 Entity 분리 시  자식 Entity도 자동으로 분리됨

> LOCK(잠금 작업 전파)

- cascade = CascadeType.LOCK
- 부모 Entity 잠금 시  자식 Entity도 자동으로 잠금됨

> REPLICATE(복제 작업 전파)

- cascade = CascadeType.REPLICATE
- 부모 Entity 복제 시  자식 Entity도 자동으로 복제됨

<br />

> Example Code ( Cascade )

**부모 Entity인 Test의 저장, 삭제 작업이 이루어지면 자식 Entity인 Example도 작업을 전파 받음**
```java
@Entity
public class Test{

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @OneToMany(mappedBy = "post", casacade = {CascadeType.PERSIST, CascadeTpe.REMOVE})
    private List<Example> example = new ArrayList<>();
}
```

<br />

---
## Cascade 사용 시 주의할 점

<br />

- 무한 루프, 스택 오버플로우와 같은 문제가 발생하지 않도록 해야 함
- 필요하지 않은 작업에 Cascade 사용 시 성능 저하가 발생할 수 있음
- Cascade 사용 시 자식 Entity에 의도하지 않은 결과가 발생하지 않도록 해야 함

- Cascade되는 Entity와 Cascade를 설정하는 Entity의 LifeCycle이 동일하거나 비슷해야 함
- Cascade되는 Entity가 Cascade를 설정하는 Entity에서만 사용되어야 함

<br />

---
## One-to-Many & Many-to-One

<br />

> One-to-Many

- One쪽의 Entity에서 Cascade를 설정함
- One쪽의 Entity가 저장, 업데이트, 삭제되면 연관된 Many쪽의 Entity에도 동일한 작업이 전파됨

<br />

> Many-to-One

- Many쪽의 Entity에서 Cascade를 설정함
- Many쪽의 entit가 저장, 업데이트, 삭제될 때 One쪽의 Entity에 어떤 작업을 전파할지 결정할 수 있음 

<br />

---
## OrphanRemoval(고아 엔티티 제거)

<br />

- 부모 Entity에 @OneToMany 또는 @OneToOne을 사용하여 관계가 설정되있음
- 자식 Entity에 @ManyToOne 또는 @OneToOne을 사용하여 부모 Entity와의 관계가 설정되있음
- 부모 Entity에서 관계 필드에 orphanRemoval = true 옵션 추가

- 부모 Entity와의 관계가 끊어진 자식 Entity는 자동으로 제거하는 옵션
- 부모-자식 관계의 일관성을 유지하고 불필요한 data를 삭제함


<br />

> Example Code ( Cascade & OrphanRemoval )



**자식 Entity가 부모와의 관계가 끊어지면 자동으로 삭제됨**
```java
@Entity
public class Parent {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Child> children = new ArrayList<>();

}

@Entity
public class Child {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    private Parent parent;

}
```