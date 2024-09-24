---
title: "Limit of Offset-Based Pagination and Solution"
excerpt: "[Spring] No-Offset and Covering Index"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2024-09-24
---

> ### Offset Pagination

  - #### Issue

    - OFFSET 값 만큼 Row를 읽은 후 LIMIT 값만큼의 Row를 가져오는 방식
      - DB는 OFFSET 이후의 데이터만을 바로 가져올 수 없다.
      - OFFSET 값이 클 경우 많은 Row를 불러와서 심각한 성능 저하 문제 발생한다.
    - LIMIT 10 OFFSET 30000000
      - 앞에서 읽었던 Row를 다시 읽어야 한다.
      - 30000010개의 Row를 불러와서 10개만 가져온다.
    - 데이터의 잦은 변경이 이루어질 때 데이터 누락과 중복이 발생할 수 있다.
      - 실시간으로 빠르게 데이터가 변경되는 SNS 같은 서비스에는 치명적인 오류가 발생할 수 있다.


  - #### Image

    ![image info](/assets/img/offset.png)
    <img src="/assets/img/offset.png" alt="" width="0" height="0">

  - #### Solution

    - No-Offset
      - OFFSET을 사용하지 않는 방식
      - 성능적으로 가장 우수
    - Covering Index
      - OFFSET을 사용하되 Index가 적용된 Column만을 활용하는 방식
      - No-Offset에 비해서는 다소 오래 걸림


<br />

---

> ### No-Offset

  - Index를 통해서 원하는 데이터에 바로 접근하는 기술
  - 마지막 조회된 Id에 대한 조건으로 다음 n개의 데이터를 응답하는 방식
  - OFFSET을 사용하는 대신 WHERE을 사용함으로써 성능을 개선한다.
    - WHERE절에 여러 조건이 들어가면 OFFSET보다 성능이 안좋다.
  - 조회 시작 부분을 Index로 빠르게 찾아서 매번 첫페이지만 읽도록 한다.
    - 페이지가 뒤로 가더라도 처음 페이지를 읽는 것과 동일한 효과를 가지게 된다.


  - ### Cursor Pagination(Keyset Pagination)

    - Cursor가 가리키는 레코드로부터 일정 개수만큼 가져오는 방식
    - 데이터 정렬의 쿼리문 작성에 있어서 유의해야 한다.
      - 복잡한 정렬 쿼리를 작성할 경우 복합키를 통해 유니크한 값을 가지게 한다.
      - 최신순 정렬 -> id와 reg_dt를 합친 복합키
      - id와 함께 사용함으로써 중복되는 상황을 방지한다.
    - 정렬이 필요없고 대용량 데이터의 추가, 삭제가 자주 일어나는 도메인(페이스북, 인스타 등 SNS)에 적합하다.

  - ### Compare
    ![image info](/assets/img/Pagination_Compare.png)
    <img src="/assets/img/Pagination_Compare.png" alt="" width="0" height="0">


  - #### Condition

    - **WHERE에 사용되는 Column에 반드시 Index가 적용**되야 된다.
      - 정확한 Paging 결과를 위해서 WHERE 절에 사용될 Column은 유니크한 값으로 구성한다.
      - Index를 사용하지 않았을 때 OFFSET보다 성능이 안 좋을 수 있다.

  - #### SQL

    ```sql
    CREATE INDEX [INDEX명] ON [TABLE명(`COLUMN명`)]

    SELECT *
    FROM [TABLE명] 
    WHERE PK < 마지막 데이터 PK
    ORDER BY PK DESC
    LIMIT [출력개수]
    ```

  - #### Code(JPA-QueryDsl)

    - 기존 Offset 기반의 pagination을 No-Offset 기반의 pagination으로 개선합니다.
    - **동적 쿼리 ltBoardId()**
      - 첫 페이지 호출 할 경우 board_id를 전달할 수 없기 때문에 별도의 로직을 통해 처리합니다.
      - boardId가 첫 페이지일 경우 null 반환을 통해 조건 통과
      - boardId가 첫 페이지가 아닐 경우 **boardId < 마지막 데이터의 boardId** 조건 실행

    - Original(개선 전)
      ```java
      public List<BoardPaginationDto> paginationOffsetBoard(String title, int pageNo, int pageSize){
        return queryFactory
          .select(
            Projections.fields(
              BoardPaginationDto.class,
              board.id.as("boardId"),
              board.title,
              board.content,
              board.regDt
            )
          )
          .from(board)
          .where(
            board.title.like(title + "%")
          )
          .orderBy(board.id.desc())
          .limit(pageSize)
          .offset(pageNo * pageSize)
          .fetch();
      }
      ```

    - No-Offset(개선 후)
      ```java
      public List<BoardPaginationDto> paginationNoOffsetBoard(Long boardId, String title, int pageSize){
        return queryFactory
          .select(
            Projections.fields(
              BoardPaginationDto.class,
              board.id.as("boardId"),
              board.title,
              board.content,
              board.regDt
            )
          )
          .from(board)
          .where(
            ltBoardId(boardId),
            board.title.like(title + "%")
          )
          .orderBy(board.id.desc())
          .limit(pageSize)
          .fetch();
      }

      private BooleanExpression ltBoardId(Long boardId){
        return boardId == null ? null : board.id.lt(boardId);
      }
      ```

  - #### Limit

    - 비즈니스 기획상 반드시 OFFSET을 사용해야 하는 경우
      - OFFSET : 페이지 번호, 페이지 사이즈 방식
      - NO-OFFSET :더보기(More) 방식
    - WHERE에 사용되는 기준 Key가 중복이 가능할 경우
      - 정확한 결과를 반환할 수 없음
    - N번째 Row를 한 번에 조회해야 하는 경우

<br />

---

> ### Covering Index

  - **SELECT, WHERE, ORDER BY, LIMIT, GROUP BY**에서 사용되는 모든 Column이 포함된 Index
  - Query를 충족시키는데 필요한 모든 데이터를 갖고 있는 Index
  - 실제로는 SELECT를 제외한 나머지 Column에 대하여 우선으로 수행한다.

  - #### Image

    ![image info](/assets/img/noOffset.png)
    <img src="/assets/img/noOffset.png" alt="" width="0" height="0">

  - #### Feature

    - Covering Index에서 모든 데이터를 검색할 수 있기 때문에 추가적으로 테이블을 조회할 필요가 없어짐
      - 디스크 I/O 작업 감소(쿼리 속도 향상)
    - Index의 크기가 커질 수 있다.
      - Index 저장을 위한 공간이 늘어나고 Index 유지를 위한 비용이 늘어난다.


  - #### Using Index

    - Covering Index 적용 시 실행 결과의 필드에 다음과 같이 표기된다.
    - Covering Index
      - Extra Field : **"Using Index"**
      - Key Field : **Index명**
    - Index Condition Pushdown
      - Extra Field : **"Using Index Condition"**

  - #### SQL
    ```sql
    CREATE INDEX [INDEX명] ON [TABLE명(`COLUMN명`), TABLE명(`COLUMN명`), TABLE명(`COLUMN명`)]

    SELECT * FROM [테이블명] AS t1
    	JOIN (
    		[커버링 인덱스를 사용해 페이징하는 서브쿼리]
    	) AS t2
    ON t1.ID = t2.ID;
    ```


<br />

---



### Reference


- https://jojoldu.tistory.com/529?category=637935
- https://taegyunwoo.github.io/tech/Tech_DBPagination
- https://bestsu.tistory.com/98
- https://giron.tistory.com/131
- https://insanelysimple.tistory.com/362
- https://velog.io/@januaryone/No-Offset-%EC%BF%BC%EB%A6%AC%EB%A1%9C-%EB%8C%93%EA%B8%80-%EA%B8%B0%EB%8A%A5-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0