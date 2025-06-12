---
title: "Resolution process using ApplicationContextProvider between spring bean and static"
excerpt: "[Spring] About Spring Bean and static"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2025-06-11
---

### 들어가면서
  - 최근 프로젝트를 진행하면서 Util에서 사용하는 static method와 Bean의 사용에 있어서 적합한 방식의 판단이 필요했던 상황이 있었습니다
  - 해당 상황의 원인/개념/고려사항/해결방법를 포스트에 남겨보고자 합니다

<br />

---

- ### ⚠️ Situation

  - CustomUtil에서 static으로 선언된 `checkValidUser()`라는 method가 존재
  - `static`으로 선언되어 있어서 Spring bean을 직접 주입해서 사용할 수 없는 상황
    - static 변수는 Spring이 의존성을 주입하기도 전에 접근될 수 있음
      - static 변수는 JVM이 class를 처음 로딩할 때 메모리에 올라감
      - Spring Bean은 Spring이 ApplicationContext를 초기화한 이후에 생성
    - static은 DI의 대상이 아님
      - DI : instance level
      - static : class level
  - Util을 호출하기 전 부분에서 해당 bean을 호출해서 사용하면 되지만 해당 Util 부분을 사용하는 부분들이 다수 존재해서 Util 내에서 처리하기로 결정
  - 일부 static method에서 Spring bean을 사용해야 하는 상황이 일부 존재

- ### 💡 Consideration
  1. `static 제거 및 CustomUtil을 Spring Bean으로 생성 -> commonService을 주입받아서 사용 가능`
      - 이미 해당 Util에서 static 형식으로 이미 많은 곳에서 사용 중이기 때문에 instance로 바꾸기에는 부담이 큼
      - 해당 방식은 다른 부분에 영향(수정, 변경)을 많이 줄 가능성 / 위험성이 있음 ❌
  2. `static method 안에서 Bean을 가져오는 ApplicationContextProvider Util class 활용`
      - `commonService`를 전역에서 접근 가능한 방식으로 `ApplicationContext`에 등록해서 사용
      - 해당 방식은 1번 방식보다 다른 부분에 영향(수정, 변경)을 거의 주지 않음 ✅
    
  - #### Explantion
    - ApplicationContext는 모든 Bean을 관리하는 Container
    - ApplicationContextAware를 구현한 `ApplicationContextProvider`는 Spring이 시작될 때 자동으로 `applicationContext`를 주입받음
      - `applicationContext`는 Spring이 모든 Bean을 초기화한 이후에 사용 가능
      - applicationContext가 null이면 `NullPointerException` 발생
    - getBean()을 통해 applicationContext에서 직접 Bean을 가져옴

- ### ✅ Solution

  - #### ApplicationContextProvider.java

    ```java
    @Component
    public class ApplicationContextProvider implements ApplicationContextAware {

      private static ApplicationContext applicationContext;

      @Override
      public void setApplicationContext(ApplicationContext context) throws BeansException {
          applicationContext = context;
      }

      public static <T> T getBean(Class<T> requiredType) {
          return applicationContext.getBean(requiredType);
      }

      public static Object getBean(String beanName) {
          return applicationContext.getBean(beanName);
      }
    }

    ```

  - #### CustomUtil.java

    ```java
      public static boolean checkUser(HashMap map) throws CustomException {
        
        boolean isValidUser = false;
	    	try {
	    		// ApplicationContext에서 빈 주입
	    		CommonService commonService = ApplicationContextProvider
                                .getApplicationContext()
                                .getBean("commonService", CommonService.class);

	    	}catch(Exception e) {
	    		e.printStackTrace();

	    	}
	    	return isValidUser;
      }

    ```