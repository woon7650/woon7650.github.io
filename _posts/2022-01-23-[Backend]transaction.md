---
title:  "[Spring]Transaction(트렌섹션)에 대하여"
excerpt: "About Transaction..."


categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2022-01-23
---

## Spring에서 Transaction의 처리(ACID의 유지)
- class 또는 method 에  @Transactional을 선언하여 사용합니다.  
자바에서 transaction을 시작하는 방법

- JDBC의 기본적인 Transaction 처리 방법
    ```java
    Connection connection = dataSource.getConntection();
    
    try(connection){
        connection.setAutoCommit(false);
    
        connection.commit();
    }catch(Exception e){
        connection.rollback();
    }

    connection.setTransactionIsolation(Connection.TRANSACTION_READ_COMMITTED);
      //TRANSACTOIN ISOLATION LEVEL 설정
    Savepoint savePoint = connection.setSavepoint();
    connection.rollback(savePoint);
    //savepoint : nested propagation option
    ```    

- Spring의 Transaction 처리 방법(선언적 트랜잭션 방식)
    ```java
    public class exampleService{
        @Transactional
        public String insertExample(Example example){
        //business code
        return seq;
        }
    }
    ```
    ```java
    @Configuration
    @EnableTransactionManagement
    public class springTransaction{
      @Bean
      public PlatforTransactionManager ptManager(){
          return ptManager;  
      }  
    }
    ```
  - @Transaction이 달린 public method에 대해 내부적으로 데이터베이스 transaction 코드를 실행합니다.
---