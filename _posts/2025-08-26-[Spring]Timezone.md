---
title: "Implementing Automatic Timezone Conversion Using MyBatis Interceptor"
excerpt: "[Spring] About Timezone Implementation"
date: 2025-08-26
categories: [Java, MyBatis, Timezone, Interceptor]
tags: [Java, MyBatis, Interceptor, Timezone, ThreadLocal, JUnit]
---



### 들어가면서
  - 서버(예: Asia/Seoul)와 사용자(클라이언트)의 타임존이 다를 경우 DB에 저장되는 시간과 화면에 보여지는 날짜/시간이 어긋나는 문제를 일관되게 해결하기 위해 Request → Controller → MyBatis Interceptor → DB 흐름에서 Timezone을 처리하는 전략을 정리하고자합니다
  - 핵심은 TimezoneInterceptor에서 SELECT/INSERT/UPDATE 시점에 Server ↔ User 타임존 변환을 수행하는 것입니다

---

- ### ⚠️ Situation

  - 해외 지사 이용이 추가될 예정으로 `duedate`, `startdate`, `regdata` 등 시간 데이터에 **시차 문제가 발생 우려**
  - 해외에서 이용 시에도 동일한 시간으로 인식되어야 하며 사용자별 Timezone 차이를 흡수할 필요가 있음
  - 이미 다수의 SQL 쿼리와 프론트엔드 코드에서 시간 데이터를 직접 사용하고 있어 전면적인 수정은 현실적으로 어려운 상황
  - 따라서 **기존 조회/수정 로직에 영향을 최소화**하면서 Timezone을 적용할 방법이 필요했음
  - map의 모든 value를 순회하면서 datetime인 부분을 변환하는 것은 **성능적으로 문제가 될 수 있으며 String으로 받아지는 경우도 있기 때문에 보류**

---

- ### 💡 Consideration
  - DB 서버의 시간대는 Asia/Seoul 기준으로 운영 중
  - 해외 사용자의 경우 `LocalDateTime` 입력/조회 시 각 사용자 Timezone으로 자동 변환되어야 함
  - MyBatis는 SQL 실행 전/후에 Interceptor를 통해 데이터를 가공할 수 있는 구조 제공
  - **MyBatisInterceptor**에는 session으로 접근이 불가능 → request마다 **ThreadLocal**을 이용해서 Timezone 파악 예정
  - Timezone을 적용할 column들을 관리 → map의 key가 column에 부합되는 부분만 빠르게 찾아서 변환 적용
    - Set 기반 column의 contains()를 통해 **빠르게 변환에 해당하는 부분인지 확인**
  - 변환 과정에서 속도 문제 발생 시 추가적으로 **stream 병렬 처리**로 Timezone 변환 추후 고려
    - List Map 형태의 수만 건이상의 대용량 데이터를 다루는 프로젝트에서는 **parallelStream** 처리
    - 외부 List 개수가 많을 경우 : `parallelStream().map(...).collect(Collectors.toList())`
      - 순서 보장이 필요
    - Map Key가 많고 내부 연산이 복잡할 경우  : `map.entrySet().parallelStream().forEach(...)`
      - 내부의 값은 서로 결과에 영향을 주지 않기 때문에 독립적 연산 가능
      - 순서 보장 필요 없음

---

- ### ✅ Solution
  - MyBatis Interceptor를 이용해 DB ↔ 애플리케이션 계층 간 시간 변환을 처리
    - SELECT 시: 서버 시간(Asia/Seoul) → 사용자 Timezone(Local)
    - INSERT/UPDATE 시: 사용자 Timezone(Local) → 서버 시간(Asia/Seoul)
  - ThreadLocal을 활용하여 사용자별 Timezone을 전달
    - 각 요청(Request)이 고유한 Thread 내에서 실행되므로 안전하게 Timezone 데이터 전달 가능
    - Filter에서 특정 조건으로 사용자 Timezone 설정
      - 로그인 시 사용자가 선택하는 Timezone
      - properties에서 지정해놓은 default Timezone
    - 사용자 Timezone 변환 완료 후 제거(메모리 누수 방지)
  - 지정한 Format의 Datetime이 아닐 경우(Input type 불일치)
    - Exception 발생하는 것이 아닌 받은 데이터를 그대로 흘림
    - Timezone을 타지 않더라도 잘못 들어온 데이터이기 때문에 다른 부분에서 Exception 발생하도록 흘려보냄

---

- ### Timezone 
  - Timezone은 특정 지역의 표준 시간을 의미하며 서버와 사용자 간 시간대가 다를 수 있음
  - 안전한 시간 데이터 처리를 위해 서버에서는 UTC로 저장하고 조회 시 사용자의 시간대로 변환하는 전략을 사용함
  - Java에서는 ZoneId와 ZonedDateTime을 활용해 서버, UTC, 사용자 시간대 간 변환을 쉽게 구현할 수 있음

- ### Architecture

  - Filter, ThreadLocal, MyBatisInterceptor, .properties, TimezoneUtil

  - #### ThreadLocal

    - `ThreadLocal`은 각 쓰레드별로 독립된 저장 공간을 제공하는 Java API
    - 다중 사용자 환경에서 동시에 요청이 들어와도 사용자별 Timezone이 섞이지 않고 안전하게 참조 가능
    - 본 구조에서는:
      - Filter : 로그인/세션 기반 Timezone 설정 → ThreadLocal에 저장
      - Interceptor : DB 값 변환 시 ThreadLocal에서 Timezone 조회
      - 변환 완료 후 : ThreadLocal clear() 호출하여 메모리 누수 방지

  - #### MyBatis Interceptor
    - MyBatis Interceptor는 SQL 실행 전/후 단계에서 데이터 가공을 가로챌 수 있는 기능을 제공
    - 이를 통해 DAO/Service/Mapper 코드 변경 없이 중앙 집중식으로 Timezone 변환 로직을 삽입 가능
    - 기존 소스에 수정이 불필요하며 레거시 호환성이 뛰어남

<br />

---

- ### Process

  - View ↔ Filter(ThreadLocal) ↔ TimezoneInterceptor(ThreadLocal) ↔ MyBatis

  - #### SELECT 시 (DB → CLient)

    - 입력 포맷(DB) : yyyy-mm-dd, yyyy-mm-dd hh:mm:ss, yyyy-mm-dd hh:mm:ss.SSS
    - Interceptor는 모두 yyyy-mm-dd hh:mm:ss.SSS 형태로 변환 후 Timezone 연산 수행
    - 출력 포맷(View)

      - .properties의 DATE_COLUMNS_TIMEZONE에 속한 컬럼은 yyyy-mm-dd 형태로 전달
      - 나머지는 모두 yyyy-mm-dd hh:mm:ss.SSS로 전달

    - 예시

      - 2025-07-30 → 2025-07-30 00:00:00 → 2025-07-30 09:00:00 → View: 2025-07-30
      - 2025-07-30 18:00:00 → 2025-07-31 03:00:00 → View: 2025-07-31
      - ✅ 정확한 연산을 위해 쿼리에서 yyyy-mm-dd 대신 yyyy-mm-dd hh:mm:ss 또는 yyyy-mm-dd hh:mm:ss.SSS로 반환하는 것을 권장

    - 일부 참고
      ```java
      try {
        Optional.ofNullable((String) value)
                  .flatMap(TimezoneUtil::parseStringToDatetime)
                  .flatMap(serverDateTime → TimezoneUtil.convertToUserTimeZone(serverDateTime, userTimezone))
                  .flatMap(dateColumns.contains(key) ? (TimezoneUtil::dateToStringFormat) : (TimezoneUtil::datetimeToStringFormat)
                  ).ifPresent(result → {
                      entry.setValue(result);
                  });
      } catch (Exception e) {
        logger.warn("Timezone 변환 실패 : key = {}, value = {}", key, value, e);
      }
      ```

  - ### INSERT / UPDATE 시 (CLient → DB)

    - 입력 포맷(View): yyyy-mm-dd, yyyy-mm-dd hh:mm:ss, null(getDate())
    - Interceptor는 모두 yyyy-mm-dd hh:mm:ss.SSS 형태로 변환 후 서버 Timezone 저장
    - 출력 포맷(DB) : 날짜를 넘겨 받는지 DB의 현재 시간으로 자동으로 들어가는 지 여부가 중요

      - getDate() 처리 : Interceptor에서 현재 시간 변환 → Query 반영
      - 나머지는 모두 yyyy-mm-dd hh:mm:ss.SSS로 전달

    - 예시

      - 2025-07-30 → 2025-07-30 00:00:00.000 → 2025-07-30 09:00:00.000
      - 2025-07-30 18:00:00 → 2025-07-31 03:00:00.000
      - map.put("creationTime", null) → Interceptor → 2025-07-31 13:20:00.231
      - ✅ 정확한 연산을 위해 쿼리에서 yyyy-mm-dd 대신 yyyy-mm-dd hh:mm:ss 또는 yyyy-mm-dd hh:mm:ss.SSS로 반환하는 것을 권장

    - 일부 참고
      ```java
      try {
          //User로부터 날짜 정보를 안받는 경우(현재시간)
          if (TimezoneUtil.isEmpty(value)) {
              TimezoneUtil.datetimeMillisToStringFormat(LocalDateTime.now())
                      .ifPresent(nowDatetimeStr → {
                          entry.setValue(nowDatetimeStr);
                      });
              continue;
          }
          //User로부터 날짜 정보를 넘겨받을 경우
          Optional.ofNullable((String) value)
                  .flatMap(TimezoneUtil::parseStringToDatetime)
                  .flatMap(userDateTime → TimezoneUtil.convertToServerTimeZone(userDateTime, userTimezone))
                  .flatMap(TimezoneUtil::datetimeMillisToStringFormat)
                  .ifPresent(result → {
                      entry.setValue(result);
                  });
      } catch (Exception e) {
          logger.warn("Timezone 변환 실패 : key = {}, value = {}", key, value, e);
      }
      ```

<br />

---


- ### Exception 처리

  - Utility 레벨 (TimezoneUtil)
    - 예외 발생 → Optional.empty() 반환
    - 잘못된 값 = 무시하고 패스
  - Interceptor 레벨 (TimezoneInterceptor)

    - Interceptor 전체 실패 → logger.error 후 그대로 throw (쿼리 실패 처리)
    - 변환 실패 → logger.warn 찍고 무시 (쿼리 실행은 계속 진행)

  - 일부 참고
    ```java
    public static Optional<LocalDateTime> stringDatetimeMillisToDatetimeFormat(String strDatetime) {

        try{
            return Optional.of(LocalDateTime.parse(strDatetime, DATETIME_MILLIS_FORMATTER));
        }catch(Exception e){
            return Optional.empty();
        }

    }
    ```

<br />

---
