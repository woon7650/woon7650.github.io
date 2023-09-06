---
title:  "@Scheduled"
excerpt: "[Spring] About @Scheduled "

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2023-09-05

---
### 1. @Scheduled

> Spring에서 지원하는 방법 중에 특정 시점이나 특정 시간 간격, 혹은 정해진 시간에 실행해야 하는 작업을 <mark style="background-color:#cccccc">Scheduled</mark> 또는 <mark style="background-color:#cccccc">Batch</mark>라고 함. 사람의 개입 없이 특정 시간 간격으로 작업을 실행할 수 있는 프로세스 중 하나로 사용

- void 사용(return x)
- parameter 사용 불가 


<br />

---

##### 1.1  Dependency

<br />

build.gradle

```java
implementation 'org.springframework.boot:spring-boot-starter-web'
```

pom.xml

```java
<dependency> 
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

<br />

##### 1.2 Activate

<br />

```java
@SpringBootApplication
@EnableScheduling
public class TestApplication {

    public static void main(String[] args) {
        SpringApplication.run(TestApplication.class, args);
    }

}
```

- Scheduler 활성화를 위해 Configuration class 또는 Main class에 @EnableScheduling를 추가함

<br />

---

### 2. Scheduler Supported

- fixedDelay / fixedDelayString
- fixedRate / fixedRateString
- initialDelay / initialDelayString
- cron
- zone

<br />

##### 2.1 fixedDelay / fixedDelayString

```java
//application.properties
fixed.delay.string = 10000
```

```java
@Scheduled(fixedDelay=10000L) 
//or @Scheduled(fixedDelayString = "${fixed.delay.string}") 
public void printLogWithFixedDelay() {
    System.out.println("print complete");
    Thread.sleep(1000L);
}
```

- 해당 작업 종료 이후부터 다음 작업 실행까지의 주기(ms)
- <mark style="background-color:#cccccc">11s(10s + 1s)</mark> 간격으로 작업 실행

<br />

##### 2.2 fixedRate / fixedRateString

```java
//application.properties
fixed.rate.string =10000
```

```java
@Scheduled(fixedRate=10000L) 
//or @Scheduled(fixedRateString = "${fixed.rate.string}") 
public void printLogWithFixedRate() {
    System.out.println("print complete");
    Thread.sleep(1000L);
}
```

- 이전 작업 종료 여부와 상관없이 다음 작업 실행까지의 주기(ms)
- <mark style="background-color:#cccccc">10s</mark> 간격으로 작업 실행


<br />

##### 2.3 initialDelay / initialDelayString

```java
//application.properties
initial.delay.string = 10000
```

```java
@Scheduled(initialDelay=10000) 
//or @Scheduled(initialDelayString = "${initial.delay.string}") 
public void printLogWithFixedDelay() {
    System.out.println("print when method initially started")
}
```

- 작업을 최초 실행하기까지 초기 지연 시간 설정 주기
- <mark style="background-color:#cccccc">10s</mark> 뒤에 최초 작업 실행


<br />

##### 2.4 cron

- Format : cron = "초 분 시 일 월 요일"
  - second(0 ~ 59)
  - minute(0 ~ 59)
  - hour(0 ~ 23)
  - day of month(0-31)
  - month(0-12 or JAN-DEC)
  - day of week(0-7 or MON-SUN)
  - cron 표현식 TIP
  - ,(skip)
  - *(모든 값)
  - /(기간)
- zone : scheduling의 time zone을 설정


```java
@Scheduled(cron = "0/10 * * * * *")
public void updateNftByScheduled(){
    List<Nft> findNfts = nftService.findTerminatedNft();
    if (!findNfts.isEmpty()){
        nftBusinessLogic.getObject().updateNftStatusToPending(findNfts);
    }
}
```
<mark style="background-color:#cccccc">10s</mark> 간격으로 판매기간이 지난 nft들의 상태를 바꿔준다

<br />

---

### 3. Thread Pool

```java
@Scheduled(fixedRate = 1000)
public void firstSchedule() {
    System.out.println("firstSchedule")
}

@Scheduled(fixedRate = 1000)
public void secondSchedule() {
    System.out.println("secondSchedule")
}
```

- 원래의 Scheduled 작업은 Spring에 의해 생성된 한 개의 Thread pool에서 실행됨
- firstSchedule 작업이 모두 완료된 후에 secondSchedule 작업이 진행됨
  - Log : firstSchedule -> secondSchedule
- Thread pool을 생성해서 여러 개의 Schedule 작업을 동시에 실행 가능


<br />

---

### Reference

- https://rooted.tistory.com/12
- https://velog.io/@kenux/Spring-Boot-Scheduled-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EA%B0%84%EB%8B%A8%ED%95%9C-%EC%8A%A4%EC%BC%80%EC%A4%84%EB%A7%81

