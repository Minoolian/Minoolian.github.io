---
layout: post
title: "Spring Security & JWT"
category: tech
tags: web
image:
  path:   https://user-images.githubusercontent.com/44282342/172929238-6034fca9-91f8-4640-87ce-52307bfc1a1b.png
---

이번 포스팅은 **Spring Security와 JWT**에 대해 알아보려한다.

* unordered toc
{:toc .large-only}

## Spring Security
***

`인증(Authentication)`과 `인가(Authorization)`를 담당한다. Filter에 흐름에 따라 처리하며 Spring Security를 사용하면 개발자 입장에서 많은 코드작성 없이 용이하게 사용할 수 있다.

![Spring Security](https://user-images.githubusercontent.com/44282342/172102125-6570256c-cf2a-47e8-9b50-59ad7ee113d5.png)

### 인증과 인가

Spring Security는 인증 후에 인가를 진행한다. 인가를 진행하면서 접근권한을 확인하며 Principal은 ID, Credential은 Password로 사용하는 Credential 기반 인증 방식을 사용한다.

### SecurityContextHolder

보안 주체의 세부정보를 포함하여 응용프로그램의 **보안 컨텍스트에 대한 세부정보**가 저장된다.

### SecurityContext

**Authentication 객체**를 보관하며, SecurityContextHolder를 통해 Authentication 객체를 꺼낼수 있다.

### Authentication

**접근하는 주체의 정보와 권한이 담겨있는 인터페이스**로, SecurityContext를 통해 접근할 수 있다.

### UsernamePasswordAuthenticationToken

Authentication을 구현한 AbstractAuthenticationToken의 하위클래스이다. 해당 클래스의 **첫번째 생성자는 인증 전의 객체**, **두번째 생성자는 인증이 완료된 객체**를 생성한다.

### AuthenticationProvider

**실제 인증을 처리하는 인터페이스**로, 인증 전 Authentication을 받아 인증 완료된 Authentication 객체를 반환한다. 해당 AuthenticationProvider 인터페이스를 구현하여 **AuthenticationManager에 등록하여 사용**한다.

### AuthenticationManager

인증을 처리하는 인터페이스지만 실제로 등록된 **AuthenticationProvider에 의해 처리**된다. 인증 성공 시 인증이 성공한 객체를 생성하여 SecurityContext에 저장한다. 그리고 세션에 보관한다.

AuthenticationManager를 구현한 ProviderManager는 실제로 **인증을 진행하는 AuthenticationProvider 목록**을 가지며, 반복문을 통해 `authenticate()` 처리를 한다.

WebSecurityConfigurerAdapter를 상속한 SecurityConfig는 AuthenticationManager를 갖고있기에 사용자 정의한 CustomAuthenticationProvider를 추가할 수 있다.

### UserDetails

인증성공 시 UserDetails 객체로 UsernamePasswordAuthenticationToken을 생성한다.

### UserDetailsService

내부의 UserRepository를 주입받아 DB 조회로 처리하여 UserDetails 객체를 반환한다.

### PasswordEncoding

SecurityConfig에 패스워드 암호화에 사용될 구현체를 지정할 수 있다. (ex. BCryptPasswordEncoder)

### GrantedAuthority

**접근 주체가 가진 권한**을 의미하며, UserDetailsService에 의해 불러올수 있고, 해당 권한으로 접근 여부를 결정한다.


## JWT (Json Web Token)
***

정보를 JSON의 형식으로 안전하게 주고 받기위한 표준(RFC 7519)이다. `HMAC` 알고리즘 또는 `RSA`, `ECDSA`를 사용하여 서명할 수 있다.

구조는 간결하게 `.`으로 3개의 부분으로 나누어져 있다.

<span style="background-color: #fe295b">**Header**</span> . <span style="background-color: #e642ff">**Payload**</span> . <span style="background-color: #00b7f4">**Signature**</span>
  
### Header

HS256(HMAC SHA256) 또는 RSA 같은 Signature에 사용될 서명 알고리즘과 토큰 유형으로 구성된다.

```json
{ "alg": "HS256", "typ": "JWT"}
```

### Payload

**Claim**(엔터티 및 추가 데이터에 대한 설명)이 구성되어있다.

1\. **Registered Claim**: iss(issuer), exp(expiratin time), sub(subject), aud(audience) 등의 정보로 구성된다.  
2\. **Public Claim**: 사용자가 정의할 수 있는 클레임이다. 충돌 방지를 위해 URI로 정의해야한다.  
3\. **Private Claim**: 정보를 공유하기 위한 맞춤 Claim이다.

### Signature

인코딩된 Header, Payload와 Secret-key를 결합하여 Header에 지정한 알고리즘으로 서명을 한다.

서명이라함은 **기밀성(Confidentailty)**과 **무결성(Integrity)**을 유지할 수 있다.

```
HS256( base64UrlEncode(header) + "." + base64UrlEncode(payload), secret-key)
```

![JWT 구조](https://cdn.auth0.com/content/jwt/encoded-jwt3.png)

## Spring Security + JWT
***

Spring Security 전반을 이해하고 **JWT를 접목**시켜보자!

### 의존성 추가

```gradle
    // https://mvnrepository.com/artifact/com.auth0/java-jwt
    implementation 'com.auth0:java-jwt'
```

### Security Config

```java
// file: "SecurityConfig.java"
@Configuration
@EnableWebSecurity // 스프링 시큐리티 필터가 스프링 필터체인에 등록
@EnableGlobalMethodSecurity(securedEnabled = true, prePostEnabled = true) // @Secured annotation 활성화 (즉 controller에서 권한설정을 할 수 있다.) <= 요즘사용
                                                                          // @preAuthorize annotation 활성화 (함수 전에 권한 설정) <= 예전사용
@RequiredArgsConstructor
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    private final CorsFilter corsFilter;
    @Bean
    public BCryptPasswordEncoder encodePwd(){
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable();
        http.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS) // Session 방식 해제
                .and()
                .addFilter(corsFilter) // cors에 대한 적용할 필터설정 추가
                .formLogin().disable() // Form Login 해제
                .httpBasic().disable() // http basic 방식 해제
                .addFilter(new JwtAuthenticationFilter(authenticationManager(), jwtProperties)) // loginProcessingUrl을 사용하지 못하므로 대신 처리할 핕터 추가
                                                                                 // 로그인 시도 url은 /login
                                                                                 // AuthenticationManager를 WebSecurityConfigurerAdapter가 가지고 있다.
                .addFilter(new JwtAuthorizationFilter(authenticationManager(), userRepository, jwtProperties))
                .authorizeRequests()
                .antMatchers("/api/v1/user/**")
                .access("hasRole('ROLE_USER') or hasRole('ROLE_MANAGER')")
                .antMatchers("/api/v1/manager/**")
                .access("hasRole('ROLE_MANAGER')")
                .anyRequest().permitAll();

    }
}
```
토큰방식으로 인해 formLogin이 필요하지 않아 disable 시킨다. 이로 인해 loginProcessingUrl이 제공되지 않으므로 **따로 필터를 추가해 활성화**해주어야 한다.

**HttpBasic 방식**은 Authorizaion header에 **ID, PASSWORD**를 담아가는 방식이다. 즉 https 통신이 아니면 노출이 되기에 위험하다.

이를 대신해 Authorization header에 **Token**을 담아가는 **Bearer 방식**을 사용한다.

**인증**을 담당하는 **JwtAuthenticationFilter**와 **인가**를 담당하는 **JwtAuthorizationFilter**를 Filter에 등록한다.

### CorsConfig

```java
// file: "CorsConfig.java"
@Configuration
public class CorsConfig {

    @Bean
    public CorsFilter corsFilter() {
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        CorsConfiguration config = new CorsConfiguration();
        config.setAllowCredentials(true); // 서버응답 시 json을 자바스크립트에서 처리할 수 있게
        config.addAllowedOrigin("*"); // 모든 ip의 응답 허용
        config.addAllowedHeader("*"); // 모든 헤더의 응답을 허용
        config.addAllowedMethod("*"); // 모든 get post put delete patch 요청 허용

        source.registerCorsConfiguration("/api/**", config);
        return new CorsFilter(source);
    }
}
```

@CrossOrigin 어노테이션으로는 `인증`이 필요한 요청에 대해서는 해결이 되지 않으므로 해당 필터를 시큐리티 필터에 등록하여 설정해야 한다.

### JwtAuthenticationFilter

```java
// file: "JwtAuthenticationFilter.java"
@RequiredArgsConstructor
public class JwtAuthenticationFilter extends UsernamePasswordAuthenticationFilter {
    private final AuthenticationManager authenticationManager;
    private final JwtProperties jwtProperties;

    // login 요청 시 로그인 시도를 하는 함수
    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {

        // 1. username 및 password를 받는다.
        try {
            ObjectMapper om = new ObjectMapper();
            User user = om.readValue(request.getInputStream(), User.class);

            UsernamePasswordAuthenticationToken usernamePasswordAuthenticationToken = new UsernamePasswordAuthenticationToken(user.getEmail(), user.getPassword());

            // 2. AuthenticaionManager로 로그인을 시도하면 PrincipalDetailsService의 loadUserByUsername() 실행
            Authentication authentication = authenticationManager.authenticate(usernamePasswordAuthenticationToken);

            PrincipalDetails principalDetails = (PrincipalDetails) authentication.getPrincipal();

            // 3. PrincipalDetails를 세션에 담는다. (권한 관리를 위한 용도)
            // 권한관리를 Security에서 진행하기 때문에 편리를 위해 세션을 이용해서 권한처리를 하는 것이다.
            return authentication;
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }



    // attemptAuthentication 이후 인증이 완료되면 해당 함수 실행
    // 4. JWT토큰 생성 및 응답
    @Override
    protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authResult) throws IOException, ServletException {
        // attemptAuthentication() 결과가 authResult로 담겨온다.
        PrincipalDetails principalDetails = (PrincipalDetails) authResult.getPrincipal();

        String jwtToken = JWT.create()
                .withSubject(principalDetails.getUsername())
                .withExpiresAt(new Date(System.currentTimeMillis() + jwtProperties.getExpire())) // 만료시간 설정
                .withClaim("email", principalDetails.getUser().getEmail()) // private claim
                .sign(Algorithm.HMAC512(jwtProperties.getSecret())); // Secret-key 설정

        response.addHeader(jwtProperties.getHeader(), "Bearer "+jwtToken);
        
    }
}
```

`attemptAuthentication()` 메서드가 `/login` API요청이 오면 실행되어 json 데이터를 ObjectMapper로 받아 UsernamePasswordAuthenticationToken으로 변환한다. 이를 바탕으로 PrincipalDetailsService의 `loadUserByUsername()` 메서드로 password를 비교한 후 **authentication 객체**를 반환한다.

이후 `successfulAuthentication()` 메서드에서 해당 authentication 객체로 **JWT 토큰을 생성**하여 **Header**에 담는다.

### JwtAuthorizationFilter

```java
// file: "JwtAuthorization.java"
public class JwtAuthorizationFilter extends BasicAuthenticationFilter {
    
    private UserRepository userRepository;
    private JwtProperties jwtProperties;

    public JwtAuthorizationFilter(AuthenticationManager authenticationManager, UserRepository userRepository, JwtProperties jwtProperties) {
        super(authenticationManager);
        this.userRepository = userRepository;
        this.jwtProperties = jwtProperties;
    }

    // 인증 또는 권한요청이 올때 해당 필터 사용
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws IOException, ServletException {
        String jwtHeader = request.getHeader(jwtProperties.getHeader());

        // Token이 존재하는 지 확인
        if (jwtHeader == null || !jwtHeader.startsWith("Bearer")) {
            chain.doFilter(request, response);
            return;
        }

        // JWT 검증
        String jwtToken = request.getHeader(jwtProperties.getHeader()).replace("Bearer ", "");

        String email = JWT.require(Algorithm.HMAC512(jwtProperties.getSecret())).build().verify(jwtToken).getClaim("email").asString();

        // 서명이 정상적으로 되면 username이 담긴다.
        if (email != null) {
            User userEntity = userRepository.findByEmail(email);

            PrincipalDetails principalDetails = new PrincipalDetails(userEntity);

            // AuthenticationManager를 통한 생성이 아닌 Jwt 토큰 서명을 통해 Authentication 객체를 생성
            Authentication authentication = new UsernamePasswordAuthenticationToken(principalDetails, null, principalDetails.getAuthorities());

            // 강제로 시큐리티의 세션영역에 접근해서 Authentication 저장.
            SecurityContextHolder.getContext().setAuthentication(authentication);

            System.out.println(SecurityContextHolder.getContext().getAuthentication());

            chain.doFilter(request, response);

        }
    }
}
```

권한이 필요한 페이지에서 요청이 들어온다면 `doFilterInternale()` 메서드가 실행되어 Header의 토큰이 올바른지 확인하는 절차를 통해 올바른 토큰을 담는다.

서버에 저장한 secret-key로 토큰의 header와 payload를 **encoding**해서 토큰의 signature로 검증한다.

검증이 완료되면 **private claim에 담긴 email(username)**으로 유저정보를 조회하여 **Session에 강제 저장**한다. 그 이유는 **권한 인가**를 용이하게 하기 위해서 사용한다.

### UserDetails & UserDetailsService

```java
// file: "PrincipalDetails.java"
@Getter
public class PrincipalDetails implements UserDetails {

    private User user;

    public PrincipalDetails(User user) {
        this.user = user;
    }

    // user의 권한을 return
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        Collection<GrantedAuthority> collect = new ArrayList<>();
        user.roleList().forEach(r->
                collect.add(() -> r)
        );
        return collect;
    }

    @Override
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    public String getUsername() {
        return user.getUsername();
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

    // 휴먼회원 처리등에 사용된다.
    @Override
    public boolean isEnabled() {
        return true;
    }
}
```

```java
// file: "PrincipalDetailsService.java"
@Service
@RequiredArgsConstructor
public class PrincipalDetailsService implements UserDetailsService {
    private final UserRepository userRepository;

    // session 안의 Authentication 내부에 UserDetails를 반환한다.
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User userEntity = userRepository.findByEmail(username);
        if (userEntity != null) {
            return new PrincipalDetails(userEntity);
        }
        
        return null;
    }
}
```

**UserDetails**를 구현한 PrincipalDetails와 **UserDetailsService**를 구현한 PrincipalDetailsService 로 인증처리를 진행한다.

### 번외

```java
// file: "FilterConfig.java"
@Configuration
public class FilterConfig {

    @Bean
    public FilterRegistrationBean<MyFilter1> filter1() {
        FilterRegistrationBean<MyFilter1> bean = new FilterRegistrationBean<>(new MyFilter1());
        bean.addUrlPatterns("/*");
        bean.setOrder(0);
        return bean;
    }
}
```

Filter를 구현한 커스텀필터는 `http.addFilterBefore(new Myfilter1(), BasicAuthenticationFilter.class);` 와 같이 Security 관련 필터 전, 후로 걸어주어야 한다. 그러므로 Config에 등록할 수도 있지만 특정한 FilterConfig를 구성해서 사용할 수도 있다.

## 결론

CoCo 프로젝트의 로그인 인증, 인가를 `JWT 토큰`을 사용하기로 하면서 Spring Security의 Session 방식만을 사용해본 내겐 정말 **좋은 기회**였다. 여기에 더해 `OAuth2 방식`과 `Refresh Token 방식`을 적용할 수 있도록 심화해야겠다는 자신감이 생겼다.