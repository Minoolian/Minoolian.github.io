---
layout: post
title: "[JWT] The token was expected to have 3 parts, but got 1."
category: tech
tags: troubleshooting
---

CoCo 프로젝트 중 **JWT 검증과정에 발생한 오류**에 대해 포스팅하려 한다.

* unordered toc
{:toc .large-only}

# 👿 Problem
***

![image](https://user-images.githubusercontent.com/44282342/174126659-d8d479ed-94e5-435b-9f67-da03f22d36f6.png)

프론트단에서 `JWT 토큰 검증`을 위해 `Header에 담는 처리`를 추가하고 회원가입을 위한 API를 보내던 중 해당 오류가 발생하였다.


# 👼 Solution
***

에러의 내용이 **3 parts**가 적혀있는 것을 보아 JWT 형식에 맞지 않는 무언가 들어왔다고 판단하여 Request Header를 확인하였다. 역시나 `Authorization : Bearer undefined`이라는 **JWT 형식에 맞지 않는 토큰**을 보내주고 있었다.

따라서 `JWTAuthorizationFilter`의 JWT 토큰검증 부분에서 예외처리를 하여 JWT 형식에 맞지 않는 데이터가 들어온다면 Filter를 지나가도록 처리하였다.

```java
// file: "JWTAuthorizationFilter.java
String email = null;
try {
    email = JWT.require(Algorithm.HMAC512(jwtProperties.getSecret())).build().verify(jwtToken).getClaim("email").asString();
} catch (JWTDecodeException e) {
    chain.doFilter(request, response);
    return;
}
```

# 💡 결론
***

회원가입 로직은 권한이 필요하지 않는 페이지인데, `BasicAuthenticationFilter`를 타는 것이 의문이다. 또한 **권한이 필요하지 않은 로직의 헤더에 토큰이 담긴다**는 것 자체가 모순인 것 같아 이번주 스프린트 회의에 건의하여 개선해야겠다.

아직 **예외처리가 빈약**해서 하나하나 신경쓰지 못하고 있다. 다음주 스프린트에서는 각가지 예외처리에 대해 신경써서 추가할 예정이다.
