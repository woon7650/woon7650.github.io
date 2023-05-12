---
title:  "[SQL]쿼리 성능 개선(Improvment Performance of Query)"
excerpt: "쿼리 성능 개선(Improvment Performance of Query)"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2023-05-02
---


## 쿼리 성능 개선(Improvment Performance of Query)

---

### select

>- select * 보다는 select A, B 필요한 row만 함
>- 필요한 row만 select시 IO가 줄어들면서 Query의 성능 개선 가능

<br />

---
### like %

>- like 사용 시 %의 사용은 가급적 뒤에만 사용하는 것이 좋음
>- %A% (x), A% (o)
>- 문자열 앞에 % 사용시 ~로 시작하는 문자열을 모두 검색해서 Query 실행 시간이 증가하지만 뒤에만 붙일 때 특정 문자열 이후의 값들만 검색하면 됨

<br />

---
### group by

>- group by는 조건에 맞는 data를 추출 후 다시 재 정렬하기 때문에 정렬 이전에 데이터 양이 많아지면 Query 실행 속도가 느려짐
>- group by 이전에 필요한 data만 가져올 수 있도록 조건을 걸어주면 Query 연산 성능 개선 가능

<br />

---
### index

>- index는 어느 부분에 무엇이 존재하는지 미리 표시하는 방법(적합한 column에 index 부여)
>- index 사용 시 해당 컬럼이 unique이거나 식별 가능한 단일의 값을 사용하면 좋음(==pk기준 검색)
>- index 사용시 테이블을 모두 scan하지 않고 색인화 되어있는 file을 scan하여 Query 연산 성능 개선


<br />

---
### where & having

>- where 절이 having절보다 먼저 실행되므로 where 절에서 나오는 data를 최소화 시킬수록 Query 연산 속도가 개선됨

<br />

---

### Subquery

>- 실행 순서 : main query -> sub query
>- 조회되는 data가 많아질수록 쿼리 성능이 현저히 떨어짐

---

#### Scalar Subquery

>- select 절에 포함된 sub query(스칼라 서브 쿼리로부터 나오는 결과는 반드시 하나여야 함)
>- 조건 data 결과값이 동일한 경우가 많을 경우 캐싱 효과를 통해 쿼리 성능 향상
>- 조건 data가 지속적으로 바뀔 경우 캐싱의 효율성이 떨어져서 쿼리 성능 저하

쿼리 성능 개선 전
```sql
SELECT id, (
    SELECT count(*)
    FROM test t
    WHERE t.id = s.id
)
FROM subject s;
```

쿼리 성능 개선 후(LEFT OUETR JOIN에 인라인 뷰를 포함함)
```sql
SELECT s.id , IFNULL(t.count, 0)
FROM subject s
LEFT OUTER JOIN (SELECT COUNT(*) count, id
                  FROM test
                  GROUP BY id) t
ON t.id = s.id
```
<br />

---