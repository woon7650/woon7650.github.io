---
title: "Indexing with JPA in Spring"
excerpt: "[SQL] Query Optimization With Indexing Part2"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2024-09-14
---

> #### Index in JPA Table

  - ##### 1. Valide Entity

    - 중복도가 낮음
    - 크기가 큰 테이블

  - ##### 2. Category

    - 단일 index 설정
    - 복합 index 설정
    - Unique index 설정

<br />

---

> #### Spring Code

  - ##### 1. 단일 index

    ```java
     @Table(name = "USER", indexes = 
       @Index(name = "idx__name__email", columnList = "name, email")
     )
     ```

  - ##### 2. 복합 index
  
    ```java
    @Table(name = "USER", indexes = {
      @Index(name = "idx__name__email", columnList = "name, email"),
      @Index(name = "idx__age__birthday", columnList = "age, birthday")
    })
    ```

  - ##### 3. Unique index

    unique=true 옵션을 추가해주면 Unique Index로 생성된다.
    unique=false가 default

    ```java
    @Table(name = "USER", indexes = {
      @Index(name = "idx__name__email", columnList = "name, email"),
      @Index(name = "idx__age__birthday", columnList = "age, birthday", unique = true)
    })
    ```

<br />

---


### Reference


- https://taronko.tistory.com/31
- https://codenme.tistory.com/83
- https://codenme.tistory.com/m/158
- https://soobysu.tistory.com/115
- https://codenme.tistory.com/83