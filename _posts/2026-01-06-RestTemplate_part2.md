---
title: "Scalable Data Transmission with RestTemplate(Part 2)"
excerpt: "[Project Experience] 고성능 I/O를 위한 엔진 최적화와 RestTemplateConfig 설계"
date: 2026-01-06
categories: [Java, Spring, Network]
tags: [RestTemplate, HttpClient, ConnectionPool, Buffer, Java, SpringBoot, Optimization]
---

## Index

1. [들어가면서](#들어가면서)
2. [❌ Issue(Problem Definition)](#-issueproblem-definition)
3. [🔁 Consideration(Approach)](#-considerationapproach)
4. [💡 TroubleShooting(Config)](#-troubleshootingconfig)
5. [✅ Conclusion](#-conclusion)

---

### 들어가면서
이전 포스트에 이어서 이번 포스트에서는 대용량 파일 전송 환경에서 시스템 전반의 안정성을 결정짓는 `RestTemplateConfig` 최적화 전략에 대해 정리하고자 합니다.
목표 : 데이터의 크기에 상관없이 항상 일정한 성능과 안정성 보장

---

### ❌ Issue(Problem Definition)

대용량 Streaming 환경에서 기본 설정을 사용할 때 발생하는 대표적인 문제
* **Connection Starvation:** Streaming 통신은 일반 API보다 Connection 점유 시간이 길어 Pool이 부족할 경우 WAS 전체의 스레드 차단으로 이어짐
* **I/O Mismatch:** 애플리케이션의 처리 단위와 네트워크 엔진의 버퍼 사이즈가 일치하지 않아 발생하는 불필요한 컨텍스트 스위칭 및 CPU 오버헤드
* **Timeout 관리 부재:** 대용량 파일 전송 중 무한 대기 상태에 빠져 시스템 자원이 고갈되는 리스크

---

### 🔁 Consideration(Approach)

#### 1. PoolingHttpClientConnectionManager의 역할
동시 다발적인 대용량 전송이 발생할 경우 Connection을 미리 확보하고 재사용함으로써 Handshake 비용을 절감하고 시스템의 생존성을 높임

#### 2. ConnectionConfig와 Buffer Alignment
애플리케이션 레이어(Part 1)에서 설정한 **8KB(8192 byte)** 처리 단위와 엔진 레이어의 버퍼 사이즈를 일치시켜 데이터 복사 비용을 최소화

#### 3. Resource Management
`setBufferRequestBody(false)` 설정을 통해 RestTemplate이 내부적으로 메모리 Buffering을 수행하지 않도록 강제하여 OOM의 근본 원인을 제거

---

### 💡 TroubleShooting(Config)

엔진 레벨의 버퍼 풀과 Connection 풀을 확보하는 Custom 설정(해당 값들은 예시를 위한 임시 값)

```java
@Configuration
public class RestTemplateConfig {

    @Bean
    public RestTemplate restTemplate() {

        ConnectionConfig connectionConfig = ConnectionConfig.custom()
                .setBufferSize(8192)      
                .setFragmentSizeHint(8192) 
                .build();

        PoolingHttpClientConnectionManager connectionManager = new PoolingHttpClientConnectionManager();
        connectionManager.setMaxTotal(200);         
        connectionManager.setDefaultMaxPerRoute(100);
        connectionManager.setDefaultConnectionConfig(connectionConfig);

        CloseableHttpClient httpClient = HttpClients.custom()
                .setConnectionManager(connectionManager)
                .setKeepAliveStrategy(DefaultConnectionKeepAliveStrategy.INSTANCE) 
                .build();

        HttpComponentsClientHttpRequestFactory factory = new HttpComponentsClientHttpRequestFactory(httpClient);
        factory.setBufferRequestBody(false); 
        factory.setConnectTimeout(5000); 
        factory.setReadTimeout(30000);    

        return new RestTemplate(factory);
    }
}
```

---

### ✅ Conclusion
- What I learned : RestTemplateUtil(Part 1)에 더해 RestTemplateConfig(Part 2)을 통해 엔진 레벨의 버퍼 사이즈와 Connection 풀을 정교하게 제어함으로써 시스템의 예측 가능성을 더 높일 수 있었습니다.
---
