---
title:  "[Vue]Mixins(믹스인)"
excerpt: "About Vue Mixins"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2023-01-08
---

## Mixins

---

### Mixins이란?


>-Vue.js는 component 단위로 이루어진 Framework<br />
>-Mixins은 Vue의 component들에 공통적으로 사용되는 부분들을 모아놓은 객체<br />
>-component들의 동일한 코드를 중복으로 작성하는 것을 방지<br />
>-재사용가능한 methods의 집합<br />
>-메소드를 모듈화시켜 중앙 통제형으로 분리


---

### Vue에서의 실제 사용

<br />

**src/main.js**
```javascript 
import Common from '@/mixins/Common.vue'

Vue.mixins(Common)
```

**src/mixins/common.vue**
```javascript 
methods:{
  cropString(str){
    if(str.length >= 9){
      return str.substr(0,9)+"...";
    }
  }
}
```

**src/views/A.vue**
```javascript 
mounted:{
  var result = this.cropString("Mixins1Example")
  //Mixins1Ex.....
}
```


**src/views/B.vue**
```javascript 
mounted:{
  var result = this.cropString("Mixins2Example")
  //Mixins2Ex.....
}
```
>A,B에 공통적으로 사용되는 method를 mixins로 분류해서 모듈화하여 사용

---

### Mixins의 생성 시점
- component의 created전에 생성됨
- mixins과 component에 같은 이름의 method가 있다면 component에 등록되어 있는 method가 동작함

---

### Mixins을 사용하면서
- 하나의 페이지 안에 여러 개의 component로 이루어져 있을 때 mixins이 여러 번 생성됨
- mixins에 jquery 사용 시 script가 여러 번 실행되서 swiper, dropbox 같은 부분에서 실행이 제대로 안되는 부분이 발생함


기존 mixins
```javascript
commonFn : function(){
  if ($(".filter_area").size() > 0) {
    $(".filter_div").each(function () {
      $(this)
        .find(".btn")
        .bind("click")
        .on("click", function () {
          if (!$(this).parents(".filter_div").hasClass("on")) {
            $(this).parents(".filter_div").addClass("on");
            $(this).siblings(".con").stop(true, true).slideDown();
          } else {
            $(this).parents(".filter_div").removeClass("on");
            $(this).siblings(".con").stop(true, true).slideUp();
          }
        });
    });
  }
}
``` 

변경된 mixins
```javascript
commonFn : function(){
  if ($(".filter_area").size() > 0) {
    $(".filter_div").each(function () {
      $(this)
        .find(".btn")
        .unbind("click")
        .bind("click")
        .on("click", function () {
          if (!$(this).parents(".filter_div").hasClass("on")) {
            $(this).parents(".filter_div").addClass("on");
            $(this).siblings(".con").stop(true, true).slideDown();
          } else {
            $(this).parents(".filter_div").removeClass("on");
            $(this).siblings(".con").stop(true, true).slideUp();
          }
        });
    });
  }
}
```

>**unbind()를 이용해서 전에 중복 실행된 mixins들의 script는 초기화하고 마지막 mixins만 적용되는 목적으로 사용**