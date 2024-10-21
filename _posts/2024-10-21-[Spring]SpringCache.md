---
title: "Improvement Of Response Speed With Spring Cache"
excerpt: "[Spring] Improvement Of Response Speed With Spring Cache"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2024-10-21
---

##### RELATED POST
- [Caching Strategy](blog/Spring-CachingStrategy/)


> ### Caching Strategy

  - 이전 포스트에서 정리한 것 처럼 RDB에서의 연산을 처리할 때마다 디스크 I/O가 발생한다.
  - 자주 사용하거나 변동이 적은 데이터를 Caching을 사용해서 RDB에 접근을 최소화, 네트워크 지연 시간 감소, 서버의 부하를 분산시킬 수 있다.
  - Decision
    - Local Cache vs Global Cache
    - Caching Strategy(Read/Write)

  ![image info](/assets/img/cache.gif)
  <img src="/assets/img/cache.gif" alt="" width="0" height="0">





<br />

---

> ### Spring Cache

  - Spring은 일부 데이터를 미리 Memory 저장소에 저장하고 저장된 데이터를 다시 읽어 사용하는  Cache 기능을 제공한다.
  - Cache Abstraction(캐시 추상화)
    - 다양한 Cache Provider와 상호작용하는 코드를 작성하는 프로세스를 단순화하고 표준화하기 위한 개념이다.
    - 개발자는 Cache Provider의 구체적인 구현에 신경쓰지 않고 필요에 따라 다양한 Cache Provider를 사용할 수 있다.
    - Cache 기술을 지원하는 Cache Manager를 Bean으로 등록해야 한다.
  - Spring에서는 **CacheManager**를 Bean으로 등록하여 지정한 저장소의 Cache를 관리한다.
  - Cache Manager(캐시 매니저)
    - Cache Instance를 관리하고 Cache에 접근할 수 있는 방법을 제공한다.
    - Spring은 **CacheManager** Interface를 제공하며 이를 구현한 다양한 Cache Manager가 존재한다.
    - 각 Cache Provider에 맞는 Cache Manger를 구성할 수 있다.

  - AOP기반의 Annotation을 제공하여 간편하게 Cache 기능을 Application에 도입할 수 있다.
    - @Cacheable, @CachePut, @CacheEvict, @Caching


  - dependency(build.gradle)
    ```java
    implementation 'org.springframework.boot:spring-boot-starter-cache'
    implementation 'org.springframework.boot:spring-boot-starter-data-redis'
    implementation 'com.github.ben-manes.caffeine:caffeine'
    ```

  - application.properties
    ```java
    spring.data.redis.host= = 127.0.0.1
    spring.data.redis.port = 6379
    ```

<br />

---

> ### Cache Manager

  - ConcurrentMapCacheManager: JRE에서 제공하는 ConcurrentHashMap을 캐시 저장소로 사용할 수 있는 구현체(**Default Spring Cache**)
  - SimpleCacheManager: 기본적으로 제공하는 캐시가 없다. 사용할 캐시를 직접 등록하여 사용하기 위한 캐시 매니저 구현체
  - EhCacheCacheManager: Java에서 유명한 캐시 프레임워크 중 하나인 EhCache를 지원하는 캐시 매니저 구현체
  - CaffeineCacheManager: Java 8로 Guava 캐시를 재작성한 Caffeine 캐시 저장소를 사용할 수 있는 구현체(EhCache와 함께 인기 있는 매니저인데, 이보다 좋은 성능을 갖는다고 한다.)
  - JCacheCacheManager: JSR-107 표준을 따르는 JCache 캐시 저장소를 사용할 수 있는 구현체
  - RedisCacheManager: Redis를 캐시 저장소로 사용할 수 있는 구현체
  - CompositeCacheManager: 한 개 이상의 캐시 매니저를 사용할 수 있는 혼합 캐시 매니저

<br />
---


> ### Look-Aside/Write-Around Caching Strategy

  - 보통 Spring Local-Memory를 다룰 때 범용적으로 가장 많이 사용하는 전략

  - #### Look-Aside(Read)

    ![image info](/assets/img/LookAside.png)
    <img src="/assets/img/LookAside.png" alt="" width="0" height="0">

    - Cache에 데이터 존재
      - Cache에서 데이터 바로 반환한다.
    - Cache에 데이터 미존재
      - DB에서 데이터를 조회해서 Cache에 저장한다.


  - #### Write-Around(Write)

    ![image info](/assets/img/WriteAround.png)
    <img src="/assets/img/WriteAround.png" alt="" width="0" height="0">

     - 데이터를 저장할 때 Cache를 거치지 않고 DB 디스크 쓰기 작업만 이루어지도록 한다.

<br />

---

> ### @EnableCaching

  - **@EnableCaching**을 사용하여 Caching을 활성화힌다.
  - Spring Caching 기능을 활성화하여 Caching 관련 Annotation을 사용할 수 있다.

  - #### Architecture

    ![image info](/assets/img/EnableCache.png)
    <img src="/assets/img/EnableCache.png" alt="" width="0" height="0">

<br />

---

> ### @Cacheable

  - Cache 저장소에 데이터를 저장하거나 조회하는 기능을 사용할 수 있다.
    - 주로 읽기 작업이 일어나는 메서드에 적용된다.
  - 해당 Annotation이 정의된 메서드를 실행하면 Cache 데이터 유무를 확인한다.
  - key는 SpEL을 사용해서 옵션/속성을 선택할 수 있다.
  - **GetMapping + @Cacheable**은 자동으로 Look-Aside 일기 전략으로 동작한다.

  - #### Process
    - Cache Hit : 메서드를 실행하지 않고 Cache에서 해당 데이터를 꺼내서 바로 반환햔다.
    - Cache Miss : DB에서 읽어온 데이터를 반환하고 Cache에 저장한다.

  - #### Optional Element
    - cacheNames : Cache 이름
    - value : CacheName의 Alias
    - key: Cache 이름이 같을 때 사용되는 구분값

  - #### Key/Value
    - Cache의 Value 값은 필수로 지정, Key값은 선택적으로 적용 가능하다.
    - Key값을 비어있을 때 메서드의 파라미터 변수명이 Key값으로 등록된다.
    - 파라미터가 존재하지 않을 경우 Key값은 0으로 처리된다.(파라미터가 없는 메서드)

<br />

---

> ### @CacheEvict

  - 데이터 추가/변경에 따른 Cache 메모리의 **Refresh**를 위한 Annotation
  - 지정된 key에 해당하는 모든 Cache를 모두 삭제한다.
  - Cache 대상 리소스에 변경 작업을 하는 메서드라면 적용해주는 것이 좋다.
    - 데이터의 변경이 일어난다면 기존 캐시 데이터를 제거해야 한다.
    - 데이터 불일치가 발생한다.(일관성 불일치)
  - 적용하지 않고 데이터 수정할 경우 DB는 수정되고 Cache 데이터는 수정 전의 상태로 남는다.

  - **allEntries = true** : Cache에 저장된 값을 모두 제거할 필요가 있을 경우

  - #### Process
    - 일정한 주기로 Cache를 제거
    - 값이 변할 때 Cache를 제거

<br />

---

> ### @CachePut

  - 메서드 실행을 방해하지 않고 Cache 내용을 업데이트 할 수 있다.
  - 메서드가 실행되고 결과가 Caching된다.
  - 업데이트하면 DB, Cache에 동시에 적용되고 메서드 실행 후 반환되는 값을 Cache에도 적용해준다.

  - #### Process
    - Cache Hit : 메서드를 실행하고 Cache에서 해당 데이터를 꺼내서 바로 반환햔다.

<br />

---

> ### @Caching

  - 동일한 Caching Annotation을 여러 개 그룹화하여 사용할 수 있다.
  - @Cacheable, @CacheEvict, @CachePut을 함께 사용할 수는 없다.


<br />

---

### Reference

- https://chagokx2.tistory.com/98
- https://1-7171771.tistory.com/138
- https://loosie.tistory.com/806
- https://tussle.tistory.com/1104
- https://jiwondev.tistory.com/282
