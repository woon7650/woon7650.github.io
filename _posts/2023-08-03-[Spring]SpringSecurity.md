---
title:  "Spring Security"
excerpt: "[Spring] About Spring Security..."

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2023-08-03

---

## 0. 들어가면서


#### Spring Security ??

> Spring Security는 강력하고 사용자 정의가 매우 용이한 인증 및 액세스 제어 프레임워크입니다. 이는 Spring 기반 애플리케이션의 보안을 위한 사실상의 표준입니다. Spring Security는 Java 애플리케이션에 인증과 인증을 모두 제공하는 데 중점을 둔 프레임워크입니다.

<br />

#### Authentication / Authorization

- Principal & Credential pattern : username & password
- Authentication 제출 -> 검증 -> Authorization 부여  


<br />

#### 용어 설명

- Authentication : 사용자가 본인이 맞는지 확인하는 절차
- Authorization : 인증된 사용자가 요청한 자원에 접근 가능한지 결정하는 절차
- Principal : 요청한 자원에 접근하는 대상
- Credential : 자원에 접근하는 대상의 비밀번호
- SecurityContext : Authentication 객체를 보관, 관리, 불러오는 역할

<br />

#### Flowchart

![image info](/assets/img/springsecurity.png)
<img src="/assets/img/springsecurity.png" alt="" width="0" height="0">


<br />

---

### 1. HTTP Request Receive

- Browser(Client)로 부터 request를 받음
- id, password를 기반으로 인증, 권한 부여를 진행
- Application Filter 중에 Authentication Filters에 가장 먼저 도달

<br />

---


### 2. AuthenticationFilter

- AuthenticationFilter가 인증, 권한 부여를 위한 과정을 먼저 거침
- 위의 과정을 마친 후 Dispatcher Servlet으로 요청을 넘김

<br />

---

### 3. UsernamePasswordAuthenticationToken

- Authentication Interface의 구현체
- Principal-Credential pattern을 반영한 객체
- 생성자를 통해서 인증 전의 Authentication 객체, 인증 후의 Authentication 객체를 생성
- 모든 접근 주체는 Authentication을 생성하고 SecurityContext에 보관 및 사용

<br />


- AbstractAuthenticationToken implements Authentication
- UsernamePasswordAuthenticationToken extends AbstractAuthenticationToken
  
<br />

Authentication Code
```java
public interface Authentication extends Principal, Serializable {

    //인증된 사용자의 권한 목록
    Collection<? extends GrantedAuthority> getAuthorities();

    //사용자 비밀번호
    Object getCredentials();
    
    //사용자 상세정보
    Object getDetails();
    
    //사용자 아이디
    Object getPrincipal();
    
    //사용자 인증 여부
    boolean isAuthenticated();
    
    void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException;
}
``` 

<br />

UsernamePasswordAuthenticationToken Code
```java
public class UsernamePasswordAuthenticationToken extends AbstractAuthenticationToken {
    
    //사용자 아이디
    private final Object principal;

    //사용자 비밀번호
    private Object credentials;
    
    //인증 완료 전의 객체 생성
    public UsernamePasswordAuthenticationToken(Object principal, Object credentials) {
	    super(null);
		  this.principal = principal;
		  this.credentials = credentials;
		  setAuthenticated(false);
	  }
    
    //인증 완료 후의 객체 생성
    public UsernamePasswordAuthenticationToken(Object principal, Object credentials, Collection<? extends GrantedAuthority> authorities) {
		  super(authorities);
		  this.principal = principal;
	    this.credentials = credentials;
	    super.setAuthenticated(true);
	}
}
```

<br />

AbstractAuthenticationToken Code
```java
public abstract class AbstractAuthenticationToken implements Authentication, CredentialsContainer {
}
```

<br />

---

### 4. AuthenticationManager

- authenticate 메소드를 제공하는 interface
- Authentication 객체를 받아 인증하고 성공 시 Authentication 객체를 return
- AuthenticationManager에 등록된 AuthenticationProvider에 의해 인증 처리가 이루어짐

<br />

- 인증 성공 시 인증이 성공한 Authentiction 객체를 생성하여 SecurityContext에 저장
- 인증 실패 시 AuthenticationException을 발생시킴

<br />

AuthenticationManager Code
```java
public interface AuthenticationManager {

	Authentication authenticate(Authentication authentication) throws AuthenticationException;

}
```
<br />

---
### 5. ProviderManager(implements AuthenticationManager)

- AuthenticationManger의 구현체
- Spring Security가 관리하는 Bean
- ProviderManager는 여러 AuthenticationProvider를 순회하면서 인증 처리가 가능한 AuthenticationProvider를 찾아서 인증 처리 과정 위임

<br />

ProviderManager Code
```java
public class ProviderManager implements AuthenticationManager, MessageSourceAware,InitializingBean {
    public List<AuthenticationProvider> getProviders() {
	  	return providers;
	  }
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
		  Class<? extends Authentication> toTest = authentication.getClass();
		  AuthenticationException lastException = null;
		  Authentication result = nul;
	    boolean debug = logger.isDebugEnabled();

      //AuthenticationProvider 중에 가능한 provider에서 result를 얻어낸다
		  for (AuthenticationProvider provider : getProviders()) {
			try {
				result = provider.authenticate(authentication);

				if (result != null) {
					copyDetails(authentication, result);
					break;
				}
			}
			catch (AccountStatusException e) {
				prepareException(e, authentication);
				throw e;
			}
            ....
		}
		throw lastException;
	}
}
```
<br />


SecurityConfig에서 직접 customize한 provider를 등록할 수 있음(빈 등록 후 ProviderManager에 AuthenticationProvider로 등록)
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
  
    @Bean
    public AuthenticationManager getAuthenticationManager() throws Exception {
        return super.authenticationManagerBean();
    }
      
    @Bean
    public customizeAuthenticationProvider customizeAuthenticationProvider() throws Exception {
        return new customizeAuthenticationProvider();
    }
    
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.authenticationProvider(customizeAuthenticationProvider());
    }
}
```

<br />


---

### 6. AuthenticationProvider

- AuthenticationProviders 중에 실제 검증을 수행하는 AuthenticationProvider
- principal을 바탕으로 해당 유저의 정보를 조회
- UserDetailService 구현체를 사용하여 유저의 정보가 담겨있는 객체를 반환 

<br />

- 인증 성공 시 인증하기 전의 Authentication 객체를 받아서 인증이 완료된 Authentication 객체로 반환
- 인증 실패 시 AuthenticationException 발생

<br />

AuthenticationProvider Code
```java
public interface AuthenticationProvider {

	//인증 전의 Authenticaion 객체를 인증된 Authentication 객체로 반환
  Authentication authenticate(Authentication var1) throws AuthenticationException;

  boolean supports(Class<?> var1); 
}
```

<br />

---

### 7. UserDetailsService

- DB에서 유저 정보를 불러와서 유저의 정보가 담겨있는 객체를 만듬

<br />

UserDetailService Code
```java
public interface UserDetailsService {

  UserDetails fetchUserDetailByUserId(String userId) throws UsernameNotFoundException;

}
```

<br />

UserDetail Code
```java
public interface UserDetails extends Serializable {

  Collection<? extends GrantedAuthority> getAuthorities();

  String getPassword();

  String getUsername();

  boolean isAccountNonExpired();

  boolean isAccountNonLocked();

  boolean isCredentialsNonExpired();

  boolean isEnabled();
    
}

```

<br />

---
