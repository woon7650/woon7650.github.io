---
title:  "[Backend]JPQL에 대하여"
excerpt: "About Java Persistence Query Language...."

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2022-09-25
---


##  JPQL(Java Persistence Query Language) 이란?
<br />

### 정의 
- Table이 아닌 Entity 객체를 조회하는 객체지향쿼리


### 특징
- 테이블이 아닌 객체를 검색하는 객체 지향 쿼리
- JPA는 JPQL을 분석하여 SQL 생성후 DB에서 조회
- SQL을 추상화해서 특정 데이터베이스 SQL에 의존하지 않음

---

### 문법

- 대소문자 구분 : 엔티티와 속성은 대소문자를 구분함

- 엔티티 이름 : JPQL에서 사용하는 것은 클래스명이 아닌 엔티티명(name 속성 생략시 default로 클래스명 사용)
```java
    @Entity(name="entityname")
```
- 별칭(alias) : JPQL에서 엔티티의 별칭은 필수적으로 명시해야 함

<br />

## TypedQuery & Query

<br />

### TypedQuery : 반환할 타입을 지정할 수 있는 쿼리 객체

```java
EntityManager em;
TypedQuery<MyInfo> query = em.createQuery(jpql, MyInfo.class);
```

<br />

### Query : 반환할 타입을 명확하게 지정할 수 없는 쿼리 객체

```java
EntityManager em;
Query query = em.createQuery(jpql);
```

> EntityManager 객체에서 createQuery() 메소드를 호출<br />
> TypedQuery : 두번째 인자 존재 / Query : 두번째 인자 없음<br />
> query.getResultList() : 결과가 없을 경우 빈 collection return<br />
> query.getSingleResult() : 결과가 정확히 하나일 때 사용
>- 결과가 없을 때 : javax.persistence.NoResultException 발생
>- 결과가 1개보다 많을 때 : javax.persistence.NoUniqueResultException 발생

---
<br />

## Parameter Binding
<br />

### Binding By Parameter Name
```java
String jpql = "select i from MyInfo i where i.name =: name"
TypedQuery<MyInfo> query = em.createQuery(jpql, MyInfo.class);
query.setParameter("name", parameter)
```
콜론(:)을 사용하여 jpql상에 지정한 파라미터를 바인딩함

<br />

### Binding By Parameter Location
```java
String jpql = "select i from MyInfo i where i.name = ?1"
TypedQuery<MyInfo> query = em.createQuery(jpql, MyInfo.class);
query.setParameter("name", parameter)
```
물음표(?)을사용하여 파라미터의 위치 값으로 바인딩함(Minimum 1)

---
<br />

## Using DTO(new)
```java
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class MyInfoDto{
    private String name;
    private Integer age;
    private Long phone;
}
```


select from 사이에 new 키워드 + dto의 패키지명을 사용함
<br />

패키지명을 포함한 전체 클래스명 기입 & 순서와 타입이 일치하는 생성자 필요

```java
String jpql = "select new com.example.MyInfoDto(i.name, i.age, i.phone) from MyInfo i"
TypedQuery<MyInfo> query = em.createQuery(jpql, MyInfo.class);
query.setParameter("name", parameter)
List<MyInfo> list = query.getResultList();
```
new: 객체 생성 x -> JPQL에서 지원하는 new

---

<br />

## Paging API
```java
setFirstResult(int start); //조회 시작 위치
setMaxResults(int amount); //조회할 데이터 수


TypedQuery<MyInfo> query = em.createQuery(jpql, MyInfo.class);
query.setFirstResult(30); //31번부터 조회 시작
query.setMaxResults(30); //31 ~ 60번 조회
```
JPA에서 제공하는 페이징 API

---
<br />


## JPQL & QueryDSL

![image info](/assets/img/querydsl.png)
<img src="/assets/img/querydsl.png" alt="" width="0" height="0">

### QueryDSL의 장점
- 파라미터 바인딩을 자동 해결해줌
- 자바 코드로 이루어져있어서 컴파일 단계에서 에러를 잡을 수 있음

<br />

### QueryDSL >> JPQL
- 동적 쿼리, 단순 쿼리일 때

<br />

### JPQL & QueryDSL 함께 사용하는 경우
- QueryDSL용 Custom Repository 필요
