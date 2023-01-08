---
title:  "[Vue]'java.lang.String' 형식의 값을 역직렬화할 수 없습니다."
excerpt: "Cannot deserialize value of type `java.lang.String` from Object value (token `JsonToken.START_OBJECT`)"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2023-01-05
---

## Cannot deserialize value of type `java.lang.String` from Object value (token `JsonToken.START_OBJECT`)

<br/>

**ERROR CODE**
```javascript
- org.springframework.http.converter.HttpMessageNotReadableException: JSON parse error: Cannot deserialize value of type `java.lang.String` from Object value (token `JsonToken.START_OBJECT`); nested exception is com.fasterxml.jackson.databind.exc.MismatchedInputException: Cannot deserialize value of type `java.lang.String` from Object value (token `JsonToken.START_OBJECT`)
```

---

### 오류의 원인

Backend
- @RequestBody의 Dto안에 "Voucher"라는 변수 타입은 String이지만 Vue에서 직렬화(JSON.stringify)가 제대로 이루어지지 않은 상태로 넘어가서 오류 발생 


Vue 
- 77자리 수의 정수는 Number를 사용할 수 없음
- Number 대신 BigInt 사용
- Json에서는 BigInt 자료형을 지원하지 않음

-> 올바른 형식의 JSON을 직렬화하지 않아서 발생한 원인

---

### JSON에서 지원하는 자료형

- Number
- String
- Boolean
- Array
- Object
- null

---

### 해결 방법


- Number, BigInt를 사용하지 않고 Dto의 자료형을 String으로 변경
- 올바른 형식의 JSON을 직렬화하고 Backend로 데이터 전달 및 오류 해결

---