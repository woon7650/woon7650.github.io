---
title: "Usage of ApplcationContextProvider"
excerpt: "[Java] ApplcationContextProvider"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2025-06-10
---

### 들어가면서


<br />

---

- ### 💡Situation

  - static Method 내부에서는 정의한 Bean이 정상적으로 주입되지 않는다

- ### 원인

  - Spring DI는 Instance Level에서 수행
  - static field