---
title: "Clustered Index and Non-Clustered Index Tuning"
excerpt: "[SQL] Query Optimization With Indexing"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2024-09-10
---

##### RELATED POST
-  [QueryImprovement](/blog/SQL-QueryImprovment/)

#### 0. 들어가면서

  > Indexes are a powerful tool used in the background of a database to speed up querying. Indexes power queries by providing a method to quickly lookup the requested data.
  Simply put, an index is a pointer to data in a table. An index in a database is very similar to an index in the back of a book.
  
  > 인덱스는 데이터베이스 백그라운드에서 쿼리 속도를 높이는 데 사용되는 강력한 도구입니다. 인덱스는 요청한 데이터를 빠르게 조회할 수 있는 방법을 제공하여 쿼리에 힘을 실어줍니다.
  간단히 말해, 인덱스는 표의 데이터에 대한 포인터입니다. 데이터베이스의 인덱스는 책 뒷면의 인덱스와 매우 유사합니다.

<br />

---

> #### Index

  - **데이터 검색 성능의 향상**을 위해 테이블 열에 사용하는 자료구조
  - 단일/여러 개의 Column을 이용하여 인덱스 생성이 가능하다.

  - ##### 1. Pros & Cons

    - Query 부하가 감소하여 시스템 전체 성능이 높아진다.
    - 인덱스 사용 시 Full Scan 없이 빠르게 검색할 수 있다.
      - 인덱스가 없으면 데이터를 찾기 위해 테이블 전체를 Full Scan하게 되어 검색 속도가 매우 느려진다.

    - 인덱스는 테이블 크기의 10% 내외의 공간을 추가로 사용한다.(사용하지 않는 인덱스는 제거해야 한다.)
    - Select Query가 아닌 INSERT/DELETE/UPDATE 작업이 자주 일어나지 않는 Column에 인덱스를 사용하는 것이 좋다.
      - DBMS는 항상최신 정렬 상태로 인덱스를 유지해야 원하는 값을 빠르게 탐색할 수 있다.
      - INSERT/DELETE/UPDATE 가 계속 수행되면 계속 정렬을 해야 해서 부하가 발생한다.
      - 부하 최소화를 위해 인덱스를 **사용하지 않음** 처리를 한다.

  - ##### 2. Correlation
  
    ![image info](/assets/img/FullScan.png)  
    <img src="/assets/img/FullScan.png" alt="" width="0" height="0">

  - ##### 3. Column

    - INSERT/DELETE/UPDATE 가 빈번하게 일어나지 않는 Column
    - Join, Where, Order by와 같은 조건문이 자주 사용되는 Column
    - 데이터 중복이 없는 Column

  - ##### 4. Category

    - Clustered Index : 인덱스와 데이터를 함께 관리한다.
    - Non-Clustered Index : 인덱스에 대한 정보를 따로 관리한다.

<br />

---

> #### Clustered Index

  - ##### 1. Features
    - B-Tree 구조를 가지는 가장 일반적인 인덱스
    - 실제 테이블 데이터가 항상 정렬된 상태를 유지하고 있고 인덱스에 실제 데이터를 저장하고 있다.
    - 가장 적합한 Column 하나만을 Clustered Index로 지정한다.
      - 테이블당 하나의 Clustered Index 존재
      - PK(Primary Key)을 지정하는 Column에 대해 자동으로 Clustered Index를 생성 
      - 적당한 PK(Clustered Index)가 없을 경우 ID 필드를 PK로 설정하고 auto_increament 옵션 사용

  - ##### 2. Pros & Cons

    - 인덱스를 가지고 Leaf Node까지 탐색해서 원하는 데이터를 바로 얻을 수 있다.
    - 인덱스 기준으로 데이터가 함께 정렬된 상태를 유지한다.
      - 검색 속도가 빠르다.
      - SELECT 외에 작업 수행 시 속도가 느리다.(재배열 필요)

  - ##### 3. Architecture

    ![image info](/assets/img/clustered.png)
    <img src="/assets/img/clustered.png" alt="" width="0" height="0">



<br />

---

> #### Non-Clustered Index

  - ##### 1. Features

    - 실제 데이터가 저장된 Heap 영역과 별도로 인덱스 페이지를 관리한다.
    - Leaf Node에서 인덱스의 RID(ROWID)를 이용하여 실제 데이터가 저장된 Heap 영역에 접근한다.
    - 인덱스 페이지만 관리하고 실제 테이블 데이터는 정렬하지 않은 상태를 유지한다.
    -  지정된 Column에 대해 여러 개의 인덱스를 생성할 수 있다.
      - unique 제약 조건을 설정한 Column에 대해 자동으로 Non-Clustered Index 생성

  - ##### 2. Pros & Cons

    - 인덱스를 가지고 Leaf Node에 탐색해서 데이터 위치만 얻고 실제 Heap 영역에 접근하여 데이터를 얻는다.
    - 인덱스만 정렬되고 실제 데이터는 정렬되어 있지 않다.
      - INSERT/UPDATE/DELETE 작업 속도가 빠르다.
      - 검색 속도가 느리다.
      
  - ##### 3. Architecture

    ![image info](/assets/img/nonclustered.png)
    <img src="/assets/img/nonclustered.png" alt="" width="0" height="0">

<br />

---

### Reference


- https://shin-bugkiller.tistory.com/42
- https://developers-haven.tistory.com/54
- https://velog.io/@juhyeon1114/MySQL-Index%EC%9D%98-%EA%B5%AC%EC%A1%B0-B-Tree-BTree
- https://developers-haven.tistory.com/55
- https://earthteacher.tistory.com/216#gsc.tab=0