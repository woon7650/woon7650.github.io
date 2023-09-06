---
title:  "@Value & static"
excerpt: "[Spring] About @Value and static "

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2023-09-06

---
### 1. @Value

> <mark style="background-color:#cccccc">.properties</mark>, <mark style="background-color:#cccccc">.yml</mark>에 설정한 내용을 주입시켜주는 Annotation. 노출되기 민감한 정보들을 모아 놓은 설정 파일을 필요한 곳에 주입시켜줌

- Format : <mark style="background-color:#cccccc">@Value("${property key}")</mark>
- 일반적으로 사용하는 상황과 static 변수로 사용할 때의 차이점이 있음

<br />

##### 1.1  .properties / .yml


```java
//application.yml
opensea:
  api :
    apikey: YOU_OPENSEA_API_KEY
    baseUrl :
      v1 : "https://api.opensea.io/api/v1"
      v2 : "https://api.opensea.io/api/v2"
    protocol : "0x00000000000000ADc04C56Bf30aC9d3c0aAF14dC"
```


<br />

##### 1.2 usage without static

```java
package com.example.utils.common;

import org.springframework.beans.factory.annotation.Value;

@Component
public class TestExample {

    @Value("${opensea.api.baseUrl.v1}")
    public String openseaV1Url;

    @Value("${opensea.api.baseUrl.v2}")
    public String openseaV2Url;

    @Value("${opensea.api.apikey}")
    public String openseaApiKey;

    @Value("${opensea.api.protocol}")
    public String openseaApiProtocol;

}
```

- 일반적으로 설정 파일에 존재하는 변수를 호출하는 방식
- @Value를 이용하여 불러오기 원하는 변수에 주입(Injecting)하도록 설정


<br />

##### 1.3 usage with static


```java
package com.example.utils.common;

import org.springframework.beans.factory.annotation.Value;

@Component
public class TestExample {

    public static String openseaV1Url;

    public static String openseaV2Url;

    public static String openseaApiKey;

    public static String openseaApiProtocol;

    @Value("${opensea.api.baseUrl.v1}")
    public void setOpenseaV1Url(String value) {
        openseaV1Url = value;
    }

    @Value("${opensea.api.baseUrl.v2}")
    public void setOpenseaV2Url(String value) {
        openseaV2Url = value;
    }

    @Value("${opensea.api.apikey}")
    public void setOpenseaApiKey(String value) {
        openseaApiKey = value;
    }

    @Value("${opensea.api.protocol}")
    public void setOpenseaApiProtocol(String value) {
        openseaApiProtocol = value;
    }
}
```

- static 변수에서 @Value 사용시 잘못된 결과를 초래할 수 있음
- TestExample.openseaApiKey로 접근시 null이 반환됨
- setter를 추가하여 static 변수에 설정 파일의 변수의 값을 넣어줌

<br />

---