---
title: "JWT Authentication with JPA"
excerpt: "[Spring] Access/Refresh Token Part 2"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2024-07-19
---

#### 0. 들어가면서

- JWT Authentication 블로그 작성 이후에 Spring에 실제로 사용되는 코드에 대해 알아보고자 함
- Refresh Token, Access Token의 Code상에서 활용부분을 좀 더 자세히 알아보고 Redis를 이용한 로그인, 로그아웃단을 구현해보고자 함
- 이번 글 이후에 jwt blacklist과 같은 추가 보안적인 부분들과 Redis를 통해서 token을 관리하는 시스템을 구현을 진행할 예정

<br />

---

### 1. JWT Authentication(Access Token, Refresh Token)

- #### 1.1 Conception
  - ##### 1.1.1 Classification

    - 인증(Authentication)
      - 로그인 요청 : ID/PW를 통한 로그인 요청
      - 정보 확인 및 JWT 생성 : 일치한다면 JWT 생성
      - JWT 발급 : 생성된 JWT 사용자에게 전달
      - JWT 저장 : 사용자는 받은 JWT 저장
      - JWT 전송 : 모든 요청에 대하여 JWT를 포함해서 전송
      - JWT 검증 : 매 요청 마다 JWT 유효성 검증
      - 요청 처리 : 유효하다면 요청 처리 후 사용자에게 결과 전달
 
    - 인가(Authorization)
      - 로그인 요청 : ID/PW를 통한 로그인 요청
      - 정보 확인 및 JWT 생성 : 일치한다면 JWT 생성
      - JWT 발급 : 생성된 JWT 사용자에게 전달
      - JWT 저장 : 사용자는 받은 JWT 저장
      - JWT 전송 : 모든 요청에 대하여 JWT를 포함해서 전송
      - JWT 검증 : 매 요청 마다 JWT 유효성 검증
      - 권한 확인 : 토큰이 유효하다면 해당 요청에 대한 권한이 있는지 확인
      - 요청 처리/거부 : 권한이 있다면 처리 없다면 거부
      
  - ##### 1.1.2 Process

    - Request -> FilterChain -> Dispatcher Servlet -> Controller -> Service

      ![image info](/assets/img/jwtFlowchart.png)
      <img src="/assets/img/jwtFlowchart.png" alt="" width="0" height="0">


- #### 1.2 Settings

  - #### 1.2.1 Dependency

    - build.gradle
    ```java
      //ver 0.12.3
      implementation 'io.jsonwebtoken:jjwt-api:0.12.3'
      implementation 'io.jsonwebtoken:jjwt-impl:0.12.3'
      implementation 'io.jsonwebtoken:jjwt-jackson:0.12.3'
    ```

  - #### 1.2.2 Global Properties

    - .yml
    ```java
    jwt: 
      secret: 
        key : [token secret key code]
    ```

    - .properties
    ```java
      jwt.secret.key = [token secret key code]
    ```


<br />

---

### 2. Implement

  - #### 2.1 TokenDto

    ```java
    @Getter
    @AllArgsConstructor
    @Builder
    public class TokenDto {
      private String grantType;
      private String accessToken;
      private String refreshToken;
    }
    ```

  - #### 2.2 TokenProvider

    ```java
    @Component
    @Transactional(propagation = Propagation.REQUIRED)
    public class JwtUtils {
    
      private final Key key;

      public static final String AUTHORITIES_KEY= "auth";
      public static final String BEARER_TYPE = "bearer";
      public static final long ACCESS_TOKEN_VALIDATION_TIME = [AccessToken 만료 기간];
      public static final long REFRESH_TOKEN_VALIDATION_TIME = [RefreshToken 만료 기간];

      public TokenProvider(@Value("jwt.secret.key") String secretKey){
        byte[] keyBytes = Decoders.BASE64.decode(secretKey);
        this.key = Keys.hmacShaKeyFor(keyBytes);
      }


      public TokenDto generateToken(Authentication authentication){
        
        String authorities = authentication.getAuthorities().stream()
          .map(GrantedAuthority::getAuthority)
          .collect(Collectors.joining(","));

        Date now = new Date();
        //User user = (User) authentication.getPrincipal();

        String accessToken = Jwts.builder()
          /*
          .setSubject(userVO.getUserId())
          .setHeader(createHeader())
          .setClaims(createClaims(userVO))
          */     
          .setSubject(authentication.getName())
          .claim("auth", authorities)                  
          .setExpiration(new Date(now.getTime() + ACCESS_TOKEN_VALIDATION_TIME))
          .signWith(key, SignatureAlgorithm.HS256)
          .compact();

        String refreshToken = Jwts.builder()
          /*
          .setSubject(userVO.getUserId())
          .setHeader(createHeader())
          .setClaims(createClaims(userVO))
          */     
          .setSubject(authentication.getName())
          .claim("auth", authorities)                              
          .setExpiration(new Date(now.getTime() + REFRESH_TOKEN_VALIDATION_TIME))
          .signWith(key, SignatureAlgorithm.HS256)
          .compact();

        return TokenDto.builder()
                      .grantType(BEARER_TYPE)
                      .accessToken(accessToken)
                      .refreshToken(refreshToken)
                      .build();
      }

      public Authentication getAuthentication(String token) {
          
          Claims claims = parseClaims(accessToken);

          if (claims.get("auth") == null) {
              throw new RuntimeException("권한 정보가 없는 토큰입니다.");
          }

          Collection<? extends GrantedAuthority> authorities 
            = Arrays.stream(claims.get("auth")
                    .toString().split(","))
                    .map(SimpleGrantedAuthority::new)
                    .collect(Collectors.toList());

          //UserDetails interface를 구현한 User Class
          UserDetails principal = new User(claims.getSubject(), "", authorities);
          return new UsernamePasswordAuthenticationToken(principal, "", authorities);
      }

      public boolean validateToken(String token) {
          try {
            Jwts.parserBuilder()
              .setSigningKey(key)
              .build()
              .parseClaimsJws(token);
            return true;
          } catch (SecurityException | MalformedJwtException e) {
              log.info(ErrorCode.FABRICATED, e);
          } catch (ExpiredJwtException e) {
              log.info(ErrorCode.EXPIRED_TOKEN, e);
          } catch (UnsupportedJwtException e) {
              log.info(ErrorCode.UNAUTHORIZED, e);
          } catch (IllegalArgumentException e) {
              log.info(ErrorCode.UNAUTHORIZED, e);
          }

          return false;
      }

      private Map<String, Object> createClaims(User user) {

        Map<String, Object> claims = new HashMap<>();

        claims.put("userId", user.getUserId());
        claims.put("userName", user.getUserName());
        claims.put("authorityId", user.getAuthorityId());

        return claims;
      }

      public Claims parseClaims(String token) {
        return Jwts.parser()
          .setSigningKey(secretKey)
          .parseClaimsJws(token)
          .getBody();
      }

      private Map<String, Object> createHeader() {
        Map<String, Object> header = new HashMap<>();
        header.put("typ", "JWT");
        header.put("alg", "HS256");
        header.put("regDate", System.currentTimeMillis());
        return header;
      }


      public UserVO getUserFromToken(String token) {
        UserVO userVO = new UserVO();
        userVO.setUserId((String) getClaims(token).get("userId"));
        return userVO;
      }
    }
    ```

    - generateToken()
      - RefreshToken, AccessToken을 생성하여 return
    - getAuthentication()
      - Token을 복호화하여 사용자의 인증 정보(Authentication) 생성
      - Process
        - Token의 "auth" Claims에서 권한 정보를 가져옴
        - 권한 정보를 보다 다양한 타입의 객체로 처리할 수 있도록 SimpleGrantedAuthority 객체로 변환하여 Collection에 추가
        - UserDetail를 생성해서 Subject(주체)와 Claims(권한 정보) 등.. 필요한 정보를 설정
        - 인증(Authentication) 객체 return
    - validateToken()
      - param으로 받은 Token의 유효성을 확인
      - 예외 처리를 통해 토큰의 유효성 판단
    - parseClaims()
      - Token 복호화 및 Claims return
      - parseClaimsJws()를 통해 Token 검증 및 파싱 수행
    - createClaims()
      - JWT 생성을 위한 Claims 생성
    - createHeader()
      - JWT 생성을 위한 Header 생성

  - #### 2.3 JwtFilter/JwtInterceptor

    - Interceptor vs Filter

      - ##### 2.3.1 Filter

        ```java
        public class JwtFilter extends OncePerRequestFilter {

          public static final String AUTHORIZATION_HEADER = "Authorization";
          public static final String GRANT_TYPE = "Bearer ";

          public JwtFilter(TokenProvider tokenProvider) {
            this.tokenProvider = tokenProvider;
          }

          private final TokenProvider tokenProvider;

          @Override
          protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
      
            String jwt = resolveToken(request);

            if (jwt != null && tokenProvider.validateToken(jwt)) {

              Authentication authentication = tokenProvider.getAuthentication(jwt);
              SecurityContextHolder.getContext().setAuthentication(authentication);

            }

            filterChain.doFilter(request, response);
          }

          private String resolveToken(HttpServletRequest request) {
            String bearerToken = request.getHeader(AUTHORIZATION_HEADER);
            if (StringUtils.hasText(bearerToken) && bearerToken.startsWith(GRANT_TYPE)) {
                return bearerToken.substring(7);
            }
            return null;
          }
        }
        ```
        - OncePerRequestFilter vs GenericFilterBean
          - GenericFilterBean
            - 기존 Filter에서 확장되서 Spring의 설정 정보를 가져올 수 있는 Abstract Class
            - API 처리 후 다른 API로 Redirect 시키는 경우 Filter가 두 번씩 적용되는 경우가 발생(중복)
          - OncePerRequestFilter
            - 사용자의 한 번의 요청당 한 번만 실행되는 필터
            - 해당 필터를 사용해서 해당 문제를 해결

        - doFilterInternal()
          - resolveToken으로 헤더에서 Token 추출
          - validateToken으로 Token의 유효성 검증
          - Token이 유효하면 SecurityContext에 인증 객체(Authentication) 저장
          - 다음 Filter로 요청 전달

        - resolveToken()
          - Request에서 다음 조건을 만족하는 Token 추출
          - 조건 
            - Header : "Authorization"
            - GrantType : "Bearer"

      - ##### 2.3.2 Interceptor

        ```java
        @Component
        public class JwtInterceptor implements HandlerInterceptor {

          @Override
          public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object   handler) throws Exception {

            String authorization = request.getHeader("Authorization");
              try {
                if (authorization != null) {
                  //logic
                }
              } catch (ExpiredJwtException e) {
                  throw new BusinessException(ErrorCode.EXPIRED_TOKEN);
              } catch (MalformedJwtException e) {
                  throw new BusinessException(ErrorCode.FABRICATED);
              } catch (BusinessException e) {
                  throw e;
              } catch (Exception e){
                  throw new BusinessException(ErrorCode.UNAUTHORIZED);
              }
            }

          @Override
          public void postHandle(HttpServletRequest request, HttpServletResponse response, Object   handler, ModelAndView modelAndView) throws Exception {
          }

          @Override
            public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object  object, Exception arg3) throws Exception {
          }
        }
        ```
      
        - Filter가 아닌 Interceptor 사용을 통해 스프링이 제공하는 기능을 사용
        - Exception Handler를 통해서 JWT의 유효성을 검증

    - Enum
    
      ```java
      public enum ErrorCode{
        UNAUTHORIZED("인증이 필요합니다."),
        FABRICATED("위조 및 변조된 토큰입니다."),
        EXPIRED_TOKEN("만료된 토큰입니다.");
        USER_NOT_FOUND("아이디 혹은 비밀번호를 다시 확인해 주세요."),

        private final String message;

        ErrorCode(final String message) {
          this.message = message;
        }

        public String getMessage() {
          return this.message;
        }
      }
      ```
      - Exception error code를 모아놓은 enum 생성


  - #### 2.4 SecurityConfig

    - 이전 글 참고
      - https://woon7650.github.io/blog/Spring-SecurityConfig/


  - #### 2.5 UserRepository(Repository)
    
    ```java

    @Repository
    public interface UserRepository extends JpaRepository<User, String> {
      Optional<User> findByUserId(String username);
    }
    ```

  - #### 2.6 User(Domain) / CustomUser(Domain)
    
    ```java
    @RequiredArgsConstructor
    public class CustomUser implements UserDetails {

      private final User user;

      @Override
      public Collection<? extends GrantedAuthority> getAuthorities() {
          Collection<GrantedAuthority> collect = new ArrayList<>();
          collect.add(() -> user.getUserAuthority().name());
          return collect;
      }

      @Override
      public String getPassword() {
          return user.getPassword();
      }

      @Override
      public String getUsername() {
          return user.getUserId();
      }

      @Override
      public boolean isAccountNonExpired() {
        return true;
      }

      @Override
      public boolean isAccountNonLocked() {
        return true;
      }

      @Override
      public boolean isCredentialsNonExpired() {
        return true;
      }

      @Override
      public boolean isEnabled() {
        return true;
      }
    
    }
    ```

    - getAuthorites()
      - User의 권한 목록을 GrantedAuthority로 변환 후 return
      - @Override method들 모두 true로 return하도록 설정

  - #### 2.7 AuthBusinessLogic

    ```java

    @Service
    @Transactional(readOnly = true)
    public class AuthBusinessLogic{

      private final UserService userService;
      private final AuthenticationManagerBuilder authenticationManagerBuilder;
      private final TokenProvider tokenProvider;

      public AuthBusinessLogic(UserService userService, TokenProvider tokenProvider, AuthenticationManagerBuilder authenticationManagerBuilder) {
        this.userService = userService;
        this.tokenProvider = tokenProvider;
        this.authenticationManagerBuilder = authenticationManagerBuilder;
      }

      public TokenDto login(UserVO userVO) {

        Authentication authentication = null;

        UsernamePasswordAuthenticationToken authenticationToken = new UsernamePasswordAuthenticationToken(userVO.getUserId(), userVO.getPassword());

        Authentication authentication = authenticationManagerBuilder.getObject().authenticate(authenticationToken);

        TokenDto token = tokenProvider.generateToken(authentication);

        return token;
      }
    }
    ```
    - login()
      - authenticationManangerBuilder.getObject().authenticate()
        - AuthenticationManager -> (구현체)ProviderManger의 authenticate() 실행
        - ProviderManger의 authenticate() -> AuthenticationProvider Interface의 authenticate() 사용
        - AuthenticationProvider Interface의 authenticate() -> UserDetailsService Interface의 loadUserByUsername() 호출
        - UserDetailsService의 loadUserByUsername() -> (구현체)CustomUserDetailsService의 loadUserByUsername() 호출

  - #### 2.8 CustomUserDetailsService
    
    ```java
    @Service
    @RequiredArgsConstructor
    public class CustomUserDetailsService implements UserDetailsService {
    
      private final UserRepository userRepository;
      private final PasswordEncoder passwordEncoder;

      @Override
      public UserDetails loadUserByUsername(String userId) throws UsernameNotFoundException {

        return userRepository.findByUserId(userId)
          .map(this::createUserDetails)
          .orElseThrow(() -> new UsernameNotFoundException("해당하는 사용자를 찾을 수 없습니다."));
      }

      private UserDetails createUserDetails(CustomUser customUser) {
        return User.builder()
          .username(customUser.getUsername())
          .password(passwordEncoder.encode(customUser.getPassword()))
          .roles(customUser.getRoles().toArray(new String[0]))
          .build();
      }
    }
    ```
    
    - loadUserByUsername()
      - user를 찾을 때 CustomUserDetailsService의 해당 메소드에서 커스텀해도 되고 AuthBusinessLogic(Service)에서 따로 선언해도 됨

<br />

---
### Reference

- https://sol-devlog.tistory.com/22
- https://velog.io/@gwon477/Spring-Security-JWT
- https://g-db.tistory.com/entry/Spring-Security-%EC%8A%A4%ED%94%84%EB%A7%81-%EB%B6%80%ED%8A%B8-Access-Token%EC%97%90%EC%84%9C-Refresh-Token%EC%B6%94%EA%B0%80%ED%95%98%EC%97%AC-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0
- https://suddiyo.tistory.com/entry/Spring-Spring-Security-JWT-%EB%A1%9C%EA%B7%B8%EC%9D%B8-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0-2
- https://gksdudrb922.tistory.com/217
- https://blogan99.tistory.com/89
- https://programmer93.tistory.com/68
- https://whatistudy.tistory.com/entry/%EC%8B%A4%EC%8A%B5-%EC%8A%A4%ED%94%84%EB%A7%81%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0-%EB%A1%9C%EA%B7%B8%EC%9D%B8%EC%B2%98%EB%A6%AC
- https://velog.io/@jyleedev/AuthenticationPrincipal-%EB%A1%9C%EA%B7%B8%EC%9D%B8-%EC%A0%95%EB%B3%B4-%EB%B0%9B%EC%95%84%EC%98%A4%EA%B8%B0