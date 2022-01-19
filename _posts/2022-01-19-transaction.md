---
title:  "Transaction(트렌섹션)"
excerpt: "transaction에 대하여"

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
    > - Active : transaction이 실행중인 상태

    > - Failed : transaction 실행에 오류가 발생하여 중단된 상태
    > - Aborted : transaction이 비정상적으로 종료되어 Rollback 연산을 수행한 상태

    > - Partially Committed : transaction의 마지막 연산 까지 실행하고 commit 연산이 실행되기 직전의 상태
    > - Committed : transaction이 성공적으로 종료되어 commit연산을 실행한 후의 상태

---

## 다수의 transaction에서 발생할 수 있는 문제

- **Dirty Read** : 다른 transaction에 의해 수정되었지만 아직 commit 되지 않은 data를 읽는 것
- **Non-Repeatable Read** : 한 transaction 내에서 같은 query를 두 번 수행했지만 그 사이에 다른 transaction 값을 수정, 삭제하여 두 query의 결과가 다르게 나타나는 현상
- **Phantom Read** : 한 transaction 내에서 같은 query를 두 번 수행했지만 첫 번째 query에서 없던 Phantom 레코드가 두 번째 쿼리에서 나타나는 현상