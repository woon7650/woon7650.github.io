---
title: "SecurityConfig"
excerpt: "[Spring] SecurityConfig.java"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2024-07-02
---


---

### 1. SecurityConfig.java

- #### 1.1 Changed
  - Spring Security 5.4이후부터 WebSecurityConfigurerAdapter가 Deprecated됨
  - void configure() -> SecurityFilterChain filterChain()으로 변경
  - 기존 방식과 다른 점은 return이 있고 Bean으로 등록해서 Component 기반의 보안 설정이 가능해짐

    - Spring Security ~5.4
      ```java
        public class WebSecurityConfig extends WebSecurityConfigurerAdapter{

          @Override
          public void configure(WebSecurity web){
            //logic
          }

          @Override
          protected void configure(HttpSecurity http) throws Exception{
            //logic
          }
        }
      ```
      - 상속한 WebSecurityConfigurereAdapter을 통해 configure()를 오버라이딩하여 설정
    - Spring Security 5.4~
      ```java
        @Bean
        public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
          //logic
        }
      ```
      - 컴포넌트 방식으로 컨테이너가 관리

- #### 1.2 Annotation

  - @Configuration
    - Bean을 수동으로 등록하기 위해 사용(의존성 주입)
      - Spring Container에서 Bean을 관리할 수 있도록 함
      - Bean 등록 시 SingleTon이 되도록 보장

  - @EnableWebSecurity
    - Spring Security 설정을 활성화함
    - filterChain이 Spring Filter Chain에 등록되어 보안 설정 적용

  - @EnableGlobalMethodSecurity
    - method 레벨에서 보안을 추가할 수 있는 Annotation
    - options
      - prePostEnable : @PreAuthorize, @PostAuthorize Annotation 활성화
      - jsr250Enabled : JSR-250 관련 Annotation 활성화
      - securedEnabled : @Secured Annotation 활성화


- #### 1.3 Process

    - SecurityConfig는 인증, 인가 관련된 세부적인 보안 기능을 설정할 수 있음

    ![image info](/assets/img/securityConfig.png)
    <img src="/assets/img/securityConfig.png" alt="" width="0" height="0">

<br />

---


### 2. Implement(Code)

- #### 2.1 Dependency

  - gradle
    ```java
    implementation 'org.springframework.boot:spring-boot-starter-security'
    ```

  - maven
    - spring
      ```java
      <dependency>
          <groupId>org.springframework.security</groupId>
          <artifactId>spring-security-config</artifactId>
      </dependency>
      ```

    - spring boot
      ```java
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
      </dependency>
      ```


- #### 2.2 Code

  WebSecurityConfigurerAdapter를 사용한 기존 방식
  ```java
  @Configuration
  @RequiredArgsConstructor
  @EnableWebSecurity
  public class SecurityConfig extends WebSecurityConfigurerAdapter {

      private final JwtAuthenticationEntryPoint jwtAuthenticationEntryPoint;
      private final JwtAccessDeniedHandler jwtAccessDeniedHandler;

      @Bean
      public PasswordEncoder passwordEncoder() {
          return new CustomPasswordEncoder();
      }

      //테스트가 원활하도록 관련 API 들은 전부 무시
      @Override
      public void configure(WebSecurity web) {
          web.ignoring().antMatchers("/test");
      }

      @Override
      protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable() // CSRF 보호 기능 비활성화
          
          //cors 재설정
          .and()
          .cors().configurationSource(corsConfigurationSource()) 
      
          //세션을 사용하지 않기 때문에 세션 설정을 Stateless 로 설정
          .and()
          .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)

          //커스텀 필터 적용
          .and()
          .apply(new CustomJwtSecurityConfig());

          // 인증 실패 처리
          .and()
          .exceptionHandling().authenticationEntryPoint(jwtAuthenticationEntryPoint)

          // 권한 실패 처리
          .and()
          .exceptionHandling().accessDeniedHandler(jwtAccessDeniedHandler)


          /*
          .exceptionHandling().authenticationEntryPoint((request, response, authException) -> {
              // 인증 실패 처리
          });
          .exceptionHandling().accessDeniedHandler((request, response, accessDeniedException) -> {
              // 권한 실패 처리
          });
          */

          //URL 인가 규칙 설정
          .and()
          .authorizeRequests() 
          .requestMatchers(CorsUtils::isPreFlightRequest).permitAll()
          // 로그인, 회원가입 API는 토큰이 없는 상태에서 요청이라 허용
          .antMatchers("/api/auth/**").permitAll()
          // 나머지 API는 모두 인증이 필요
          .anyRequest().authenticated()
          
          
      }

      @Bean
      public CorsConfigurationSource corsConfigurationSource() {
          CorsConfiguration corsConfiguration = new CorsConfiguration();

          corsConfiguration.addAllowedOriginPattern("*"); //모든 IP주소 허용
          corsConfiguration.addAllowedHeader("*"); //모든 헤더 허용
          corsConfiguration.addAllowedMethod("*"); //GET, POST, PUT ,DELETE 모두 허용
          corsConfiguration.addExposedHeader("Content-Disposition"); //헤더
          corsConfiguration.setAllowCredentials(true); // 클라이언트 쿠키 요청 허용
          UrlBasedCorsConfigurationSource urlBasedCorsConfigurationSource = new UrlBasedCorsConfigurationSource();
          urlBasedCorsConfigurationSource.registerCorsConfiguration("/**", corsConfiguration); // ** 규칙 : Header, Method, IP, 쿠키 요청 허용
          return urlBasedCorsConfigurationSource;
      }

  }


  ```

  - ##### 2.2.1 Bean
    - PasswordEncoder
    - SecurityFilterChain
    - CustomSecurityFilterChain
    - CustomAuthenticationHandler
    - CustomAuthorizeHandler

  - ##### 2.2.2 Methods

    - configure() / filterChain()

      - WebSecurity
        - 특정 요청들을 무시하고 싶을 때 사용
        - js, css, image 파일이나 API 중 Security를 적용할 필요가 없는 리소스를 설정

      - HttpSecurity
        - 세부적인 보안 기능을 설정할 수 있는 API 제공
          - URL 접근 권한 설정
          - 커스텀 로그인 페이지 지원
          - 인증후 성공/실패 핸들링
          - 사용자 로그아웃
          - CSRF 공격으로부터 보호
        - http
          - authorizeRequests() : URL 경로에 대한 인가 규칙을 설정
          - antMatchers() : 특정 URL에 대해서 권한 설정
          - permitAll() : antMatchers 설정 URL의 접근을 인증절차 없이 허용
          - anyRequest().authenticated() : 모든 요청에 대해 인증을 요구
          - hasRole() : 특정 역할을 가진 사용자만 접근을 허용
          - formLogin() : 폼 기반 로그인을 활성화
          - loginPage() : 로그인 페이지의 경로 지정
          - defaultSuccessUrl() : 로그인 성공 URL 설정
          - logout() : 로그아웃을 처리하는 설정 추가
          - logoutUrl() : 로그아웃 URL 지정
          - logoutSuccessUrl() : 로그아웃 성공 URL 설정
          - csrf() : CSRF 공격 방어 설정 활성화
          - sessionManagement() : 세션 관리 설정
          - sessionCreationPolicy() : 세션 생성 정책 설정

    

<br />

---
### Refernce

- https://velog.io/@jjya_3562/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0-SecurityConfig
- https://spring.io/guides/gs/securing-web
- https://velog.io/@hyeonjoonpark/Spring-Security-Config-%ED%8C%8C%EC%9D%BC-%EC%84%A4%EC%A0%95
- https://kimchanjung.github.io/programming/2020/07/02/spring-security-02/
- https://velog.io/@dbswns1101/Spring-Security-1-Security.config-%EB%B3%80%EA%B2%BD-%EC%A0%90
- https://okky.kr/questions/1187943
- https://cocococo.tistory.com/entry/Spring-Boot-Spring-Security-%EA%B6%8C%ED%95%9C-%EC%84%A4%EC%A0%95-%EB%B0%8F-%EC%82%AC%EC%9A%A9-%EB%B0%A9%EB%B2%95#google_vignette
- https://velog.io/@dbswns1101/Spring-Security-1-Security.config-%EB%B3%80%EA%B2%BD-%EC%A0%90