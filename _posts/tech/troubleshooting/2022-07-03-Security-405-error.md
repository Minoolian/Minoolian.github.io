---
layout: post
title: "[JWT] Authentication 예외처리 중 405 Error 발생"
category: tech
tags: troubleshooting
---

CoCo 프로젝트 중 **Authentication (Login 인증) 예외처리 중 발생했던 오류**에 대해 포스팅하려 한다.

* unordered toc
{:toc .large-only}

# 👿 Problem
***

![error 메시지](https://user-images.githubusercontent.com/44282342/177197444-7abf3144-aaea-4257-8922-c1930a67b651.PNG)

Authentication의 예외처리는 보통 `AuthenticationEntryPoint` 인터페이스를 구현하여 SecurityConfig에 ExceptionHandling 하는 방식으로 처리하는 것을 확인하였다. 마찬가지로 우리 프로젝트도 이와 같이 진행하려 하였지만, 로그인 실패 시 `405 Method Not Allowed` 메시지와 함께 `내부적으로 오류`가 발생하는 것으로 확인하였다.

# 👼 Solution
***

우리 프로젝트의 경우 두가지의 문제가 있었다.
1. ***anyRequest().permitAll()***: AuthenticationException이 발생하면 `AuthenticationEntryPoint`를 거치지 않아 `405 Method Not Allowed`가 발생한다.

2. ***anyRequest().authenticated()***: AuthenticationException이 발생하면 `AuthenticationEntryPoint`를 거쳐 정상적으로 예외처리가 된다.

디버깅을 통해 정확한 이유를 검증하려 하였지만, 시간을 너무 소비하게 되어 추후 다시 알아볼 예정이다.

```java
//file: "JWTAuthenticationFilter.java"
    @Override
    protected void unsuccessfulAuthentication(HttpServletRequest request, HttpServletResponse response, AuthenticationException failed) throws IOException, ServletException {
        toResponse(response);
    }

    private void toResponse(HttpServletResponse response) throws IOException {
        ObjectMapper om = new ObjectMapper();

        response.setContentType("application/json;charset=utf-8");
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);

        Map<String, Object> map = new HashMap<>();
        map.put("status", 401);
        map.put("code", "UNAUTHENTICATED_MEMBER");
        map.put("message", "이메일 또는 비밀번호가 올바르지 않습니다.");
        response.getWriter().write(om.writeValueAsString(map));
    }
```

결론은 `UsernamePasswordAuthenticationFilter`를 상속받은 `JWTAuthenticationFilter`에서 `unsuccessfulAuthentication()` 메서드를 상속받아 예외를 처리하는 것이다.

이 방법은 한 블로그에서 **모든 필터에서 발생하는** *AuthenticationException*을 ExceptionTransalcationFilter 즉, `AuthenticationEntryPoint`에서 처리하는 것이 아니고, `UsernamePasswordAuthenticationFilter`에서 발생하는 *AuthenticationException*은 부모 클래스인 AbstractAuthenticationProcessingFilter의 `unsuccessfulAuthentication()` 메서드에서 처리한다는 것을 보고 해결책을 생각해내었다.

# 💡 결론
***

예외처리가 생각보다 까다롭다. **RestController**를 이용한 예외처리로 모두 처리하고 싶었지만, **Filter**는 Controller단에 오기 전에 동작하므로 한번에 처리할 수 없었다.

JWT를 SpringSecurity와 결합하여 구현하였는데, **Filter를 커스터마이징** 하려다보니 분석하는데 시간을 꽤 소비하고 있다😥 **Interceptor로 구현**한 글들도 많이 본 것 같은데 추후에 꼭 구현해보고 차이점을 비교해봐야겠다.