---
layout: post
title: "[Spring Security] Test시 403 Error"
category: tech
tags: troubleshooting spring
---

Spring Security가 적용된 Controller 테스트에서 `403 Status Code` 가 발생하는 이유에 대해 포스팅하려 한다.

* unordered toc
{:toc .large-only}

# 👿 Problem
***

![with(csrf())](https://user-images.githubusercontent.com/44282342/189523424-fe7fb7ff-4871-4a1f-820c-f689d1017fe3.PNG)

Member 정보 업데이트를 위한 Put 핸들러 메서드를 테스트하던 중 403 Error가 발생하였다. 우선 403 에러가 발생하였으니 두가지의 의문을 품게되었다.

## 1. SecurityContextHolder에 Authentication이 설정이 되지 않았는가?

`@WithMockUser` 어노테이션을 통해 간단하게나마 Authentication 객체를 설정하였었다. 근본적인 해결책은 아니었지만, 이를 통해 **UserDetails를 Custom했다면** 그에 맞는 Authentication 설정을 위해 또한 **Custom Annotation을 통해 적절한 Authentication 객체를 생성**하여 설정하여야한다는 것을 알게되었다.

```java
// file: "WithMockCustomUserSecurityContextFactory.java"
public class WithMockCustomUserSecurityContextFactory implements WithSecurityContextFactory<WithMockCustomUser> {
    @Override
    public SecurityContext createSecurityContext(WithMockCustomUser annotation) {
        SecurityContext context = SecurityContextHolder.createEmptyContext();

        PrincipalDetails principalDetails = new PrincipalDetails("1234567", RoleType.USER, Collections.singletonList(new SimpleGrantedAuthority(RoleType.USER.getCode())));
        Authentication authentication = new UsernamePasswordAuthenticationToken(principalDetails, "NO_PASS", principalDetails.getAuthorities());
        context.setAuthentication(authentication);

        return context;
    }
}
```

```java
// file: "WithMockCustomUser.java"
@Retention(RetentionPolicy.RUNTIME)
@WithSecurityContext(factory = WithMockCustomUserSecurityContextFactory.class)
public @interface WithMockCustomUser {

}
```

## 2. CSRF 토큰의 문제?

Google 검색을 통해 Test 관련 403 오류들을 보면 대부분 ***CSRF토큰***에 대한 문제였다. 하지만 우선적으로 고려하지않은 이유는 `SecurityConfig` 설정에 `csrf().disable()`을 필터에 추가했기 때문이다. 하지만 결론적으로는 이 부분이 문제였다.

# 👼 Solution
***

```java
// file: "MemberControllerTest.java"
...
ResultActions actions = mock.perform(
                put("/members/{memberId}", memberId)
                        .accept(MediaType.APPLICATION_JSON)
                        .contentType(MediaType.APPLICATION_JSON)
                        .with(csrf()) // 해당 설정을 추가
                        .content(body)
        );
...
```

`.with(csrf())` 설정을 추가하여 확인해보니 Request의 Parameter에 _csrf 속성으로 값이 담겨들어오는 것을 확인하였다. 또한 403 에러도 동시에 해결되어 테스트가 정상적으로 동작하였다.

# 💡 결론
***

의문을 가졌던 `csrf().disable()`을 설정하였는데 csrf 관련 문제가 발생하는 것에 대해서는 아직 알아내지 못했다. 하지만 근본적으로 csrf 공격을 막기위해서는 csrf 토큰을 활성화시켜야하는 것이 맞고, 지금은 개발단계이기에 꺼놓은 상태라고 생각하고 우선 진행해야겠다.