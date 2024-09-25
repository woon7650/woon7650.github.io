---
title: "JPA N+1 Problem and Solutions"
excerpt: "[Spring] JPA N+1 문제와 해결방법"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2024-09-25
---


#### 0. 들어가면서

  - 이번 포스트에서는 JPA 개발 과정에서 흔히 발생하는 N+1 문제에 대해서 다뤄보고자 합니다.

  - ##### Issue

    - 연관 관계가 설정된 Entity를 조회할 경우에 조회된 데이터 갯수(n)만큼 연관 관계의 SELECT Query가 추가로 발생하여 데이터를 읽어오게 된다.

<br />

---

> ### Entity

  - 사용자는 여러 개의 아이템을 보유할 수 있다.
  - 아이템은 한 명의 유저에게 종속되어 있다.

    ```java
    //User Entity
    @Entity
    @Getter
    @Setter
    @NoArgsConstructor
    public class User{
      @Id
      @GeneratedValue(strategy = GenerationType.IDENTITY)
      private int id;

      private String name;

      @OneToMany(mappedBy = "user", fetch = FetchType.EAGER)
      private List<Item> items;
    }
    ```
    ```java
    //Item Entity
    @Entity
    @Getter
    @Setter
    @NoArgsConstructor
    public class Item{
      @Id
      @GeneratedValue(strategy = GenerationType.IDENTITY)
      private int id;

      private String name;

      @ManyToOne
      private User user;

      public Item(String name){
        this.name = name;
      }
    }
    ```

    - userService.userList()를 실행한다.
    - DB
      - 현재 10명의 User가 존재한다.
      - 각 User 별로 10개씩 Item을 보유하고 있다.

    ```java
    @Service("userService")
    @Transactional
    public class UserServiceImpl implements UserService{


      private final UserRepository userRepository;

      public UserServiceImpl(UserRepository userRepository){
          this.userRepository = userRepository;
      }

      @Override
      public List<User> userList() throws Exception{
        User user = userRepository.findAll()
          .orElseThrow(() ->  
            new BusinessException("유저가 존재하지 않습니다.")
          );
        return user;
      }
    }

    ```

    - 실행 결과
      - Item을 조회하는 Query가 User를 조회한 횟수만큼 Query가 호출된다.
      - **FetchType.Eager라서 발생하는 문제일까??**

<br />

---

> ### Lazy Loading

  - FetchType을 Eager에서 Lazy로 변경해본다.
    ```java
    //변경 전
    @OneToMany(mappedBy = "user", fetch = FetchType.EAGER)
    private List<Item> items;
    ```
    ```java
    //변경 후
    @OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
    private List<Item> items;
    ```

  - userService.userList()를 실행한다.

  - 실행 결과
    - User를 호출하는 Query 하나만 발생한다.
    - FetchType.LAZY는 연관 관계 데이터를 Proxy 객체로 바인딩하고 실제로 연관 관계 Entity를 Proxy만으로는 사용하지 않는다.
    - Item에 대한 정보를 호출하는 부분을 구현했을 때 N+1문제는 동일하게 발생한다.
    - 단지 N+1 발생 시점을 연관 관계 데이터를 사용하는 시점으로 미룬 것 뿐이다.

  - N+1 발생 이유??
    - JPQL은 JPA에서 객체지향적으로 Query를 작성할 수 있게 해주는 Query 언어
    - 실제 DB Table이 아닌 Entity 객체를 대상으로 작동한다.
    - findAll() 수행 시 **SELECT * FROM USER** Query만 수행된다.


  - Solution

    - Fetch Join
    - EntityGraph
    - FetchMode.SUBSELECT
    - Batchsize
    - QueryBuilder

<br />

---



> #### Fetch Join

  - Fetch Join을 추가한 JPQL

    ```java
    @Query("select u from User u join fetch u.items")
    List<User> findAllFetchJoin();
    ```

  - 수행 결과
    - Fetch Join을 사용하면 호출 시점에 모든 연관 관계의 데이터를 가져온다.(Eager)
    - **INNER JOIN**이 수행 된다.
    - 연관 관계에서 설정한 FetchType을 사용할 수 없다.

<br />

---

> #### @EntityGraph

  - attributePaths : Query 수행 시 바로 가져올 Field 지정

  - @EntityGraph를 추가한 JPQL
    ```java
    @EntityGraph(attributePaths = "items")
    @Query("select u from User u")
    List<User> findAllEntityGraph();
    ```
  - 수행 결과
    - Fetch Join을 사용하면 호출 시점에 연관 관계의 데이터를 가져온다.(Eager)
    - **OUTER JOIN**이 수행 된다.

  - Fetch Join, @EntityGraph 주의사항
    - 카테시안 곱(Cartesian Product)이 발생하여 User의 개수만큼 Item의 중복 데이터가 존재할 수 있다.
      - JPQL의 distinct를 사용하여 중복을 방지한다.
      - Set(LinkedHashSet)을 사용하여 중복을 허용하지 않는 Collection을 사용한다

        ```java
        @OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
        private Set<Item> items = new LinkedHashSet<>();
        ```

<br />

---

> #### FetchMode.SUBSELECT

  - 연관 관계의 데이터를 조회할 경우 Subquery로 함께 조회하는 방법

  - User에 FetchMode.SUBSELECT를 적용

    ```java
    @Fetch(FetchMode.SUBSELECT)
    @OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
    private List<Item> items;
    ```

  - 수행 결과
    - FetchType.EAGER일 경우 조회 시점에 Subquery가 실행된다.
    - FetchType.LAZY일 경우 연관 관계 데이터 조회 시점에 Subqery가 실행된다.


<br />

---

> #### BatchSize

  - Hibernate에서 제공하는 **org.hibernate.annotations.BatchSize** 사용
  - 연관된 Entity를 조회할 때 지정된 size만큼 SQL의 IN절을 사용해서 조회
  - 연관 관계 데이터의 최적화 데이터 사이즈를 알기가 어려움

  - User에 @BatchSize 적용

    ```java
    @BatchSize(size=5)
    @OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
    private Set<Item> items = new LinkedHashSet<>();
    ```


  - 수행 결과
    - User의 Id들을 모아서 SQL IN절을 수행한다.
    - FetchType.EAGER일 경우 조회 시점에 Item의 개수가 10개이므로 IN절을 2번 실행한다.
    - FetchType.LAZY일 경우 연관 관계 데이터 조회 시점에 5개를 미리 Loading하고 6번째 Entity 사용 시점에 다음 SQL IN절을 추가로 실행한다.

  - application.yml에서 Default BatchSize 설정
    ```java
    spring:
      jpa:
        properties:
          hibernate:
            default_batch_fetch_size : 
    ```
<br />

---



### Reference


- https://velog.io/@jinyoungchoi95/JPA-%EB%AA%A8%EB%93%A0-N1-%EB%B0%9C%EC%83%9D-%EC%BC%80%EC%9D%B4%EC%8A%A4%EA%B3%BC-%ED%95%B4%EA%B2%B0%EC%B1%85
- https://velog.io/@hero6027/JPA-Proxy%ED%94%84%EB%A1%9D%EC%8B%9C%EC%99%80-Lazy-Loading%EC%A7%80%EC%97%B0%EB%A1%9C%EB%94%A9
- https://jojoldu.tistory.com/165
- https://incheol-jung.gitbook.io/docs/q-and-a/spring/n+1#fetchmode.subselect
- 