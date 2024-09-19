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

  - ##### 1. Valide Table/Column

    - 규모가 있는 테이블
    - 잦은 변경이 없는 테이블
    - 중복된 값이 적은 열
    - 자주 사용하는 검색 조건

  - ##### 2. Category

    - 단일 index 설정
    - 복합 index 설정
    - Unique index 설정

<br />

---

> #### Index Setting

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


> #### Eample

  - ##### 1. Original Entity

    ```java
    @Entity
    @Getter
    @Setter
    @NoArgsConstructor
    public class FollowRequest {
    
      @Id
      @GenerateValue(strategy = GenerationType.IDENTITY)
      private Long id;

      @ManyToOne(fetch = FetchType.LAZY)
      @JoinColumn(name = "from_user")
      @JsonIgnore
      private User fromUser;

      @ManyToOne(fetch = FetchType.LAZY)
      @JoinColumn(name = "to_user")
      @JsonIgnore
      private User toUser;

      @Column(name = "STATUS")
       private String status;

    }

    ```
  - ##### 2. Solution

    - 자주 사용하는 쿼리 : Follow를 요청한 사람과 요청 받은 사람의 id 중 하나라도 일치하고 Status가 요청중인 데이터를 찾는 쿼리
    - Status는 인덱싱에 적합하지 않다.
      - Cardinality가 낮다.
      - 자주 수정될 가능성이 높다.
    - 두 개의 Column from_user와 to_user에 대한 인덱싱이 적합하다.
      - 검색조건에 존재한다.
      - Cardinality가 높다.
      - 수정될 가능성이 적다.
    - 만약 Follow 요청 거절 시 데이터가 삭제될 경우 발생하는 문제가 있지만 인덱싱이 주는 이점이 더 클 경우 인덱싱을 사용한다.
 
  - ##### 3. Entity With Index
  
    ```java
    @Entity
    @Getter
    @Setter
    @NoArgsConstructor
    @Table(indexes = {
      @Index(name = "request_index", columnList = "from_user, to_user")
    })
    public class FollowRequest {
    
      @Id
      @GenerateValue(strategy = GenerationType.IDENTITY)
      private Long id;

      @ManyToOne(fetch = FetchType.LAZY)
      @JoinColumn(name = "from_user")
      @JsonIgnore
      private User fromUser;

      @ManyToOne(fetch = FetchType.LAZY)
      @JoinColumn(name = "to_user")
      @JsonIgnore
      private User toUser;

      @Column(name = "STATUS")
       private String status;

    }

    ```

<br />

---


### Reference


- https://taronko.tistory.com/31
- https://codenme.tistory.com/83
- https://codenme.tistory.com/m/158
- https://soobysu.tistory.com/115
- https://codenme.tistory.com/83