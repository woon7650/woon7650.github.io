---
title: "Response with CustomException and CustomExceptionHandler"
excerpt: "[Spring] Custom 예외 처리를 통한 응답"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2024-08-28
---


#### 0. 들어가면서

  - Runtime Error 발생 시 모두 500 Error로 Response가 발생
  - 토이프로젝트를 진행하면서 Backend 부분의 CustomException과 CustomExceptionHandler를 추가하여 해당 이슈를 해결한 방법을 다루고자 합니다.
  - Error의 status, code, message를 통해 원인을 쉽게 파악

<br />

---

#### 1. Spring Project Architecture 
    
  - ##### 1.1 Package

    ![image info](/assets/img/CustomException.png)
    <img src="/assets/img/CustomException.png" alt="" width="0" height="0">
      

  - ##### 1.2 Components

    - ###### Exception
      - CustomException.class
        - Error Code를 Custom Exception에 추가

    - ###### Handler
      - CustomExceptionHandler.class

        - @RestControllerAdvice
          - Controller에서 전역적으로 발생하는 Error를 처리

        - @ExceptionHandler
          - 발생한 CustomException를 해당 메소드에서 처리

        - @RestControllerAdvice + @ExceptionHandler(CustomException.class)
          - 전역적으로 CustomException 발생 시 해당 메소드로 Exception을 Handle

    - ###### Enum
      - ErrorCode.enum
        - 다양한 상황에 나타날 수 있는 Error들의 Status, Code, Message들을 정해서 정의

    - ###### Response
      - ErrorResponseEntity.class
        - Error에 대한 정보를 담을 ErrorResponseEntity 생성(ErrorCode를 통해 Response를 생성)


<br />

---

#### 2. Code

  - ##### 2.1 ErrorCode

    ```java
    package com.example.login.common.enumType;

    import lombok.AllArgsConstructor;
    import lombok.Getter;
    import org.springframework.http.HttpStatus;

    @AllArgsConstructor
    @Getter
    public enum ErrorCode {
    
      //ERROR
      // 400 Bad Request
      // 403 Forbidden
      // 404 Not Found
      // 405 Method Not Allowed
      // 409 Conflict
      // 500 Internal Server Error

      BAD_REQUEST(HttpStatus.BAD_REQUEST, "B0", "잘못된 요청입니다."),

      FORBIDDEN(HttpStatus.FORBIDDEN, "F0", "권한이 없습니다."),

      USER_NOT_FOUND(HttpStatus.BAD_REQUEST, "U0", "사용자를 찾을 수 없습니다."),
      INVALID_PASSWORD(HttpStatus.BAD_REQUEST, "U1", "비밀번호가 일치하지 않습니다."),
      USER_ALREADY_EXIST(HttpStatus.BAD_REQUEST, "U2", "이미 가입한 사용자입니다."),

      METHOD_NOT_ALLOWED(HttpStatus.METHOD_NOT_ALLOWED, "M0", "허용되지 않은 메소드입니다."),

      INTERNAL_SERVER_ERROR(HttpStatus.INTERNAL_SERVER_ERROR, "S0", "서버에 오류가 발생하였습니다."),
      ;

      private final HttpStatus httpStatus;
      private final String code;
      private final String message;

      public int getHttpStatusCode() {
          return httpStatus.value();
      }

    }
    ```

    <br />

  - ##### 2.2 CustomException

    ```java
    package com.example.login.common.exception;

    import com.example.login.common.enumType.ErrorCode;
    import lombok.AllArgsConstructor;
    import lombok.Getter;

    @AllArgsConstructor
    @Getter
    public class CustomException extends RuntimeException{
      ErrorCode errorCode;
    }

    ```

    
  
  - ##### 2.3 ErrorResponseEntity

    ```java
    package com.example.login.common.response;

    import com.example.login.common.enumType.ErrorCode;
    import lombok.Builder;
    import lombok.Data;
    import org.springframework.http.ResponseEntity;

    @Data
    @Builder
    public class ErrorResponseEntity {
    
      private int status;
      private String code;
      private String message;

      public static ResponseEntity<ErrorResponseEntity> toResponseEntity(ErrorCode e){
          return ResponseEntity
            .status(e.getHttpStatus())
            .body(ErrorResponseEntity.builder()
                .status(e.getHttpStatus().value())
                .code(e.getCode())
                .message(e.getMessage())
                .build()
            );
      }
    }

    ```

    

  - ##### 2.4 CustomExceptionHandler

    ```java
    package com.example.login.common.handler;

    import com.example.login.common.exception.CustomException;
    import com.example.login.common.response.ErrorResponseEntity;
    import org.springframework.http.ResponseEntity;
    import org.springframework.web.bind.annotation.ExceptionHandler;
    import org.springframework.web.bind.annotation.RestControllerAdvice;

    @RestControllerAdvice
    public class CustomExceptionHandler {
    
      @ExceptionHandler(CustomException.class)
      protected ResponseEntity<ErrorResponseEntity> handleCustomException(CustomException e){
          return ErrorResponseEntity.toResponseEntity(e.getErrorCode());
      }
    }

    ```

  
  - ##### 2.5 Usage

    ```java
    User user = userRepository.findById(userDto.getId()).orElseThrow(() ->  new CustomException(ErrorCode.USER_NOT_FOUND));
    ```

    - 해당 user가 존재하지 않을 경우 CustomException이 발생하여 ErrorCode를 기반으로 Response를 만들어서 전달

<br />
---



### Reference

- https://github.com/woon7650/redis-backend