---
title:  "[Vue]'java.lang.String' 형식의 값을 역직렬화할 수 없습니다."
excerpt: "Cannot deserialize value of type `java.lang.String` from Object value (token `JsonToken.START_OBJECT`)"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2023-01-05
---

## async와 await

### async

```javascript
- org.springframework.http.converter.HttpMessageNotReadableException: JSON parse error: Cannot deserialize value of type `java.lang.String` from Object value (token `JsonToken.START_OBJECT`); nested exception is com.fasterxml.jackson.databind.exc.MismatchedInputException: Cannot deserialize value of type `java.lang.String` from Object value (token `JsonToken.START_OBJECT`)
 at [Source: (PushbackInputStream); line: 1, column: 561] (through reference chain: com.geneuin.flatabackend.dto.nft.NftDto["nftVoucher"])
```


---

### await
