---
title: "Scalable Data Transmission with RestTemplate(Part 1)"
excerpt: "메모리 효율과 데이터 안정성을 위한 아키텍처 이해"
date: 2026-01-05
categories: [Java, Spring, Network]
tags: [RestTemplate, HttpClient, Streaming, Java, SpringBoot, MessageConverter, JDK1.8]
---

## Index

1. [들어가면서](#들어가면서)
2. [❌ Issue(Problem Definition)](#-issueproblem-definition)
3. [⚠️ Potential Problem](#-potential-problem)
4. [🔁 Consideration(Approach)](#-considerationapproach)
5. [💡 TroubleShooting](#-troubleshooting)
6. [✅ Conclusion](#-conclusion)

---

### 들어가면서
대용량 데이터를 송수신할 경우 `RestTemplate`의 기본 설정인 **Buffering** 방식은 심각한 **Out Of Memory(OOM)** 를 유발합니다.
이를 해결하기 위해 `setBufferRequestBody(false)`를 설정하여 Stream 방식을 활성화하지만 HTTP 프로토콜의 엄격한 전송 순서와 Apache HttpClient의 제약으로 인해 예기치 못한 상황에 직면하게 됩니다. 
이번 포스트에서는 아래의 조건의 대용량 데이터 환경에서도 안정적인 RestTemplate 활용 전략에 대해 정리하고자 합니다.


- RestTemplate 기반 통신 환경
- 대용량 파일이 대다수인 경우
- RestTemplateConfig
    - PoolingHttpClientConnectionManager로 connection 확보/재사용
    - buffer Pool 확보/재사용(8KB)

---

### ❌ Issue(Problem Definition)

#### 1. 주요 증상 및 에러 로그
* `setBufferRequestBody(false)` 설정 후 `RestTemplate.execute()` 내부에서 `request.getBody()`를 직접 호출할 때 예외 발생.
* **Error:** `java.lang.UnsupportedOperationException: getBody() is not supported` 또는 `java.io.IOException: Stream closed`

#### 2. 발생 원인: 상태 머신(State Machine) 위반
Apache HttpClient Engine은 Streaming 방식에서 매우 엄격한 순서를 요구합니다.
* **정상 순서 :** Header 설정 → Engine에게 Header 전송 승인 요청 → Engine이 선로를 열고 Header 송출 → Body Stream 개방(`getBody()`) → 데이터 쓰기
* **에러 상황 :** 개발자가 Header 설정 후 Engine의 승인 절차 없이 곧바로 `getBody()`를 호출하면 Engine은 "아직 보낼 준비가 안 됐다"며 거부

---

### ⚠️ Potential Problem

* **프로토콜 불일치:** `setBufferRequestBody(false)` 환경에서는 데이터가 즉시 선로로 나가야 하므로 한 번 닫힌 Stream은 재사용할 수 없습니다.
* **멀티파트 규격 파손:** 파일 업로드 시 `Boundary` 문자열과 Header 정보를 바이트 단위로 직접 코딩할 경우, 미세한 줄바꿈(`\r\n`) 오류로 인해 서버가 요청을 거절할 위험이 큽니다.
* **리소스 누수:** 대용량 응답을 받을 때 `byte[]`나 `String`으로 담으면 Heap 메모리가 급증하여 GC 오버헤드가 발생하고 서버가 멈출 수 있습니다.

---

### 🔁 Consideration(Approach)

#### 1. Apache HttpClient Engine의 전송 메커니즘
Streaming 방식에서 `ClientHttpRequest`는 Engine과 긴밀하게 연결되어 있습니다. `MappingJackson2HttpMessageConverter`는 내부적으로 Engine의 API를 호출하여 **Header를 패킷화하여 선로에 먼저 쏘는 작업**을 수행한 뒤에야 안전하게 `getBody()`를 호출합니다.

#### 2. HttpMessageConverter의 고도화된 역할
단순한 JSON Converter가 아니라 **Engine과의 통신 프로토콜을 대신 관리**해줍니다. Converter 사용 하지 않으면 직접 해당 부분을 구현하면 됩니다.(복잡성이 높고 안정성 테스트 필요)

#### 3. 리소스 지연 로딩 (Lazy Loading) 전략
파일 업로드 시 메모리를 아끼는 핵심은 **데이터가 아닌 지도를 전달하는 것**입니다. `FileSystemResource`는 파일의 위치 정보만 가지고 있다가 Converter가 실제 전송을 시작하는 시점에만 8KB 버퍼만큼 조금씩 데이터를 퍼 올립니다.


---

### 💡 TroubleShooting

- JSON 전송: execute + Converter (세밀한 제어와 전송 순서 보장)
- 파일 업로드: exchange + FileSystemResource (복잡한 규격 자동화 및 메모리 최적화)
- 파일 다운로드: execute + Manual Streaming (OOM 방지를 위한 필수 선택)


#### 1. JSON Streaming: 직접 호출 대신 Converter 위임
직접 `request.getBody()`를 부르는 대신 Converter의 `write()` 메서드를 사용하여 Engine과의 절차를 자동화합니다.(개발 생산성)

```java
private RequestCallback createJsonRequestCallback(Object body) {
    return request -> {
        request.getHeaders().setContentType(MediaType.APPLICATION_JSON);
        request.getHeaders().setAccept(Collections.singletonList(MediaType.ALL));

        MappingJackson2HttpMessageConverter jsonConverter = 
            new MappingJackson2HttpMessageConverter(objectMapper);
        jsonConverter.write(body, MediaType.APPLICATION_JSON, request);
    };
}
```
### 5.2 파일 업로드: exchange와 FileSystemResource의 조화

복잡한 멀티파트 규격(Boundary 생성)은 Spring에 맡기고 데이터는 
FileSystemResource로 전달하여 Streaming을 보장합니다.
```java
public <T> T uploadMultiPart(String apiUrl, MultiValueMap<String, Object> parts, ParameterizedTypeReference<T> typeRef) {
    HttpHeaders headers = new HttpHeaders();
    headers.setContentType(MediaType.MULTIPART_FORM_DATA);
    HttpEntity<MultiValueMap<String, Object>> entity = new HttpEntity<>(parts, headers);

    return restTemplate.exchange(apiUrl, HttpMethod.POST, entity, typeRef).getBody();
}
```
### 5.3 대용량 다운로드: 응답 Stream의 직접 연결
서버로부터 오는 응답을 byte[]에 담지 않고 곧바로 파일 출력 Stream에 꽂아 넣습니다.

```java
public Path downloadFile(String apiUrl, Object body, String targetPath) throws Exception {
    Path savePath = Paths.get(targetPath);
    
    if (savePath.getParent() != null && Files.notExists(savePath.getParent())) {
        Files.createDirectories(savePath.getParent());
    }

    return restTemplate.execute(apiUrl, HttpMethod.POST, createJsonRequestCallback(body),
        response -> {
            try (InputStream is = response.getBody();
                 OutputStream os = Files.newOutputStream(savePath, StandardOpenOption.CREATE)) {
                StreamUtils.copy(is, os);
                return savePath;
            } catch (Exception e) {
                Files.deleteIfExists(savePath);
                throw e;
            }
        }
    );
}
```

### ✅ Conclusion
- What I learned : 라이브러리가 제공하는 추상화 이면에서 데이터가 어떻게 흐르는지 이해할 때 비로소 대용량 트래픽에도 무너지지 않는 견고한 백엔드 시스템을 설계할 수 있다고 생각합니다.
- 추가적으로 고려해야 할 부분
    - StreamUtils.copy -> heap 복사 없이 fileChannel transfer
    - Json Request Accept Type 동적으로 변경
    - RestTemplateUtil에 맞게 확장성 지향