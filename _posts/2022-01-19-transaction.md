---
title:  "[DB]Transaction(트렌섹션)에 대하여"
excerpt: "About Transaction..."

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2022-01-19
---

## Transaction이란? 

> 데이터베이스 상태를 변환시키는 하나의 논리적 기능을 수행하기 위한 작업의 단위입니다.

- 데이터베이스에서 rollback 혹은 commit을 통해 처리되는 작업의 논리적 단위입니다.

- 사용자의 요구에 따라 시스템이 응답하기 위한 상태 변환 과정의 작업단위입니다.

---

## Transaction의 특징(ACID)

- **Atomicity(원자성)** : transaction의 모든 연산/명령은 반드시 모두 완벽히 수행되거나 전혀 수행되지 않아야한다.   
*Rollback or Commit(All or Nothing)*

- **Consistency(일관성)** : 하나의 transaction이 성공적으로 완료되고 그 결과로 언제나 일관성 있는 데이터베이스 상태로 변환한다.  
*before & after transaction 고정 요소는 같은 상태*

- **Isolation(독립성)** : 하나의 transaction이 실행 중일 때 다른 transaction의 연산이 끼어들 수 없다.   
*transaction은 개별적으로 수행*

- **Durability(지속성)** : transaction이 성공적으로 완료되고 나면 해당 transaction의 결과는 향후 발생하는 장애에도 보존되어야 한다.   
*after commit transaction에 의한 모든 변경은 보존* 

---

## Transaction의 연산 및 상태

- Transaction 연산

    > - Commit : transaction 작업이 성공적으로 수행되고 데이터베이스가 다시 일관된 상태일 때 갱신 연산이 완료된 것을 transaction 관리자에게 알려주는 연산

    > - Rollback : transaction 작업이 비정상적으로 종료되어 데이터베이스의 일관성을 깨뜨렸을 때 해당 transaction이 행한 모든 연산으 취소하는 연산

- Transaction 상태

    - Active : transaction이 실행중인 상태

    - Failed : transaction 실행에 오류가 발생하여 중단된 상태
    
    - Aborted : transaction이 비정상적으로 종료되어 Rollback 연산을 수행한 상태

    - Partially Committed : transaction의 마지막 연산 까지 실행하고 commit 연산이 실행되기 직전의 상태
    
    - Committed : transaction이 성공적으로 종료되어 commit연산을 실행한 후의 상태  
     
---

## 다수의 Transaction에서 발생할 수 있는 문제

- **Dirty Read**(Uncommitted Dependency)

    - 아직 commit되지 않은 수정 중인 데이터를 다른 transaction에서 읽을 수 있도록 허용할 때 발생(해당 transaction이 rollback하는 경우)
    

- **Non-Repeatable Read**(Inconsistent Analysis)

    - 한 transaction 내에서 같은 query를 두 번 수행했지만 그 사이에 다른 transaction 값을 수정, 삭제하여 두 query의 결과가 다르게 나타나는 현상
    
- **Phantom Read**

    - 한 transaction 내에서 같은 query를 두 번 수행했지만 첫 번째 query에서 없던 Phantom 레코드가 두 번째 쿼리에서 나타나는 현상


---

## Transaction 고립화 수준  

- **Level0**(Read Uncommitted)

    - transaction에서 처리중인 아직 commit이 되지 않는 데이터를 다른 transaction이 읽는 것을 허용합니다.
    - dirty read, non-repeatable read, phantom read 현상이 모두 발생합니다.

    > `SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED`

- **Level1**(Read Committed)

    - transaction이 commit되어 확정된 데이터만 읽는 것을 허용합니다.
    - non-repeatable read, phantom read 현상이 발생합니다.

    > `SET TRANSACTION ISOLATION LEVEL READ COMMITTED`

- **Level2**(Repeatable Read)

    - 먼저 실행된 transaction이 읽는 데이터에 대해서는 다른 transaction이 데이터를 바꾸는 것을 제한하여 두 번 행해진 쿼리는 일관성 있는 결과는 보여줍니다.
    - phantom read 현상이 발생합니다.

    > `SET TRANSACTION ISOLATION LEVEL REPEATABLE READ`

- **Level3**(Serializable Read)

    - Level2 + 중간에 transaction이 새로운 레코드를 삽입하는 것을 제한합니다. 
    
    > `SET TRANSACTION ISOLATION LEVEL SERIALIZABLE` 
  
---

## Transaction의 동시성 & 일관성 

- Transaction 고립화 수준을 높이면 **일관성(consistency)** 이 향상되지만 **동시성(concurrency)** 은 저하됩니다. 
- 반대로 고립화 수준을 낮추면 **일관성(consistency)** 은 저하되고 **동시성(concurrency)** 이 향상됩니다.


---

## 마무리를 하면서 느낀점

- 웹 프로젝트를 다루면서 commit, rollback 을 자주 사용하기 때문에 *transaction* 에 대하여 자세히 공부하고 알아보는 것은 매우 도움이 되는  기회였습니다. 자세히 모르고 사용하는 것 보다 잘 알고 사용하는 것이 나중에 CODE상에서나 DB PART에서 발생할 error에 대하여 대처하는데 매우 도움이 될 것 같습니다.