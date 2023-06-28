---
title:  "Pre Post Annotation"
excerpt: "JPA About Pre & Post Annotation"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2023-05-29
---

### Pre & Post Annotation 이란?

<br />

- Entity의 Lifecycle 중에 영속화, 업데이트, 삭제 이벤트 전후에 실행되는 callback method
- Pre & Post Annotation을 통해서 Entity의 Lifecycle에서 추가적인 작업처리가 가능함
- Pre Annotation : Entity의 Lifecycle 이벤트 전에 실행되도록 하는 annotation
- Post Annotation : Entity의 Lifecycle 이벤트 후에 실행되도록 하는 annotation

<br />

---
### Pre Annotation의 종류

<br />

*@PrePersist*

- Entity가 영속화되기 전에 호출되는 method
- Entity가 Database에 저장되기 전에 추가적인 초기화 작업을 수행할 수 있음

*@PreUpdate*

- Entity가 update되기 전에 호출되는 method
- Entity가 Database에 update되기 전에 추가적인 작업을 수행할 수 있음

*@PreRemove*

- Entity가 remove되기 전에 호출되는 method
- Entity가 Database에서 remove되기 전에 추가적인 작업을 수행할 수 있음

<br />

> Example Code ( Post Annotation )

<br />

**Entity에 Persist, Update, Remove가 발생하기 전에 항상 이루어져야 되는 작업이 있을 떄 사용**
```java
@Entity
public class PreExample {
    @Id
    @GeneratedValue
    private Long id;
    
    private String content;
    
    @PrePersist
    public void prePersist() {
        System.out.println("Entity 영속화 성공")
    }
    
    @PreUpate
    public void preUpdate() {
        System.out.println("Entity Update 성공")
    }
    @PreRemove
    public void preRemove() {
        System.out.println("Entity Remoeve 성공")
    }
}
```

<br />

---
### Post Annotation의 종류

<br />

*@PostPersist*

- Entity가 영속화된 후에 호출되는 method
- Entity가 Database에 저장된 후에 추가적인 초기화 작업을 수행할 수 있음

*@PostUpdate*

- Entity가 update된 후에 호출되는 method
- Entity가 Database에 update된 후에 추가적인 작업을 수행할 수 있음

*@PostRemove*

- Entity가 remove된 후에 호출되는 method
- Entity가 Database에서 remove된 후에 추가적인 작업을 수행할 수 있음
- 
<br />


> Example Code ( Post Annotation )

<br />

**Entity에 Persist, Update, Remove가 발생한 후에 항상 이루어져야 되는 작업이 있을 떄 사용**
```java
@Entity
public class PostExample {
    @Id
    @GeneratedValue
    private Long id;
    
    private String content;
    
    @Postpersist
    public void postPersist() {
        System.out.println("Entity 영속화 전")
    }
    
    @PostUpate
    public void postUpdate() {
        System.out.println("Entity Update 전")
    }
    @PostRemoev
    public void postRemove() {
        System.out.println("Entity Remove 전")
    }
}
```

<br />

---
