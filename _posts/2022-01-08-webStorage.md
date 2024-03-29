---
title:  "Web Storage"
excerpt: "[Storage] Localstorage, Sessionstorage, Cookie"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2022-01-08
---

## Web storage
- WEB에서 지원하는 소량의 데이터를 저장하기 위한 Key/Value Storge
- Mobile : 2.5Mb, Desktop : 5~10MB
- localStorage, sessionStorage로 이루어져있습니다.

---
## SessionStorage
- 개념
    - 일시적인 저장/조회 기능을 가지는 web storage
    - Client가 브라우저 창을 닫는 순간 sessionStorage가 담는 정보가 사라집니다.
    - 동일한 browser 내에서만 유지됩니다.
- 쓰임
    - 단발성 정보에 사용하는 것이 적합합니다.
    - session이 사라지는 순간 사라지기를 원하는 정보를 저장하는 것이 localStorage를 사용하는 것보다 효율적입니다. 
- 활용 방법
    - value가 *String*인 경우
        - `sessionStorage.setItem(key,value);`
        - `localStorage.getItem(key);`
    - value가 *Array*나 *Object*인 경우(JSON 형태로 저장)
        - `sessionStorage.setItem(key, JSON.stringify(value));`
        - `JSON.parse(localStorage.getItem(key));`

---
## LocalStorage
- 개념
    - 영구적인 저장/조회 기능을 가지는 web storage
    - localStorage의 정보를 지워주지 않는 한 저장되어 있습니다.
- 쓰임
    - 페이지가 이동시에도 들고 다녀야 하는 정보들 저장에 적합합니다.(사용자 정보, 로그인해서 들고 다녀야 하는 정보들)
    - 계속 사용해야하는 정보를 일일이 parameter로 들고 다니지 않아도 됩니다.
    - client의 로그인, 로그아웃에도 정보가 남아있기 때문에 browser가 닫혀도 남아있기를 원하는 정보를 저장하는 데 적합합니다.
    - cookie의 기능을 대체하기에 좋습니다.
- 활용 방법
    - value가 *String*인 경우
        - `localStorage.setItem(key,value);`
        - `localStorage.getItem(key);`
        - `localStorage.removeItem(key);`
    - value가 *Array*나 *Object*인 경우(JSON 형태로 저장)
        - `localStorage.setItem(key, JSON.stringify(value));`
        - `JSON.parse(localStorage.getItem(key));`
    - 특정 key의 localStorage 삭제
        - `localStorage.removeItem(key);`
    - 전체 localStorage 삭제
        - `localStorage.clear();`

---
## 차이점
- 데이터의 영구성(localStorage o, sessionStorage x)

---
## 주의사항
-  web storage는 문자형 데이터 타입만 지원하기 때문에 다른 데이터 타입의 저장을 원한다면 JSON.stringify(value)를 통해서 문자열 형태로 변환 후 가져올 때 JSON.parse(value) JSON 파싱을 통해서 기존 데이터 타입으로 DATA를 가져옵니다.

---
## 비교

||localStorage|sessionStoroage|Cookie|
|:--:|:--:|:--:|:--:|
|생성|client|client|client or server|
|영구성|O|X|설정에 따라 다름|
|접근성|모든 browser|동일 tab|모든 browser|

---
## Cookie
- 정보들을 정해진 유효기간까지 사욯알 수 있도록 해주는 저장소의 한 형태입니다.
- 서버, 클라이언트 어느 쪽에서 구동시키냐에 따라서 저장되는 위치가 달라집니다.

---
## 마무리를 하면서 느낀점
- sessionStorage, localStorage의 활용을 통해서 이동시 데이터를 서버와 통신시 마다 들고 다니는 번거로움이 줄었고 영구성의 특성을 이용해서 각각 데이터 용도에 맞게 storage를 사용하면서 코딩의 효율성이 올라갔습니다. 또한 크로스사이트 스크립트 공격 취약점의 해결에도 도움이 되었습니다.
