---
layout: post
title: "[SpringBoot] AntPathMatcher Query String 매칭 문제 (+번외:가변인자)"
category: tech
tags: troubleshooting
---

Test 코드 작성 중 MockMvcResultMatchers의 **redirectUrlPattern() 메서드 관련 문제**에 대해 포스팅하려 한다.

* unordered toc
{:toc .large-only}

# 👿 Problem
***

```java
actions.andExpect(status().is3xxRedirection())
                .andExpect(redirectedUrlPattern(verifyUri))
```

OAuth2 관련 테스트에는 `Redirection`이 많아서 redirection url이 올바르게 설정되어있는지 검증하기 위한 코드를 작성 중이었다. Redirection uri가 **"https://accounts.google.com/o/oauth2/v2/auth"** 이후에 여러 `Query String`을 받는 형태이기에 **AntPathMatcher** 형식을 따르는 `redirectedUrlPattern()`의 인자에 내가 생각한 올바른 Pattern을 작성해도 통과되지 않기에 여러 케이스를 분석해보게 되었다.

# 👼 Solution
***

```java
AntPathMatcher antPathMatcher = new AntPathMatcher();

assertThat(antPathMatcher
                .match("https://accounts.google.com/o/oauth2/v2/auth", // (!)
                        "https://accounts.google.com/o/oauth2/v2/auth?response_type=code&client_id=x&scope=email%20profile&state=Emrx5OshqhaWyoV82JJ_HZNk28hipx2I3nh6dIFw1Ss%3D&redirect_uri=http://localhost:8080/login/oauth2/code/google"),
        is(false));

assertThat(antPathMatcher
        .match("https://accounts.google.com/o/oauth2/v2/auth*", // (2)
                "https://accounts.google.com/o/oauth2/v2/auth?response_type=code&client_id=x&scope=email%20profile&state=Emrx5OshqhaWyoV82JJ_HZNk28hipx2I3nh6dIFw1Ss%3D&redirect_uri=http://localhost:8080/login/oauth2/code/google"),
        is(false));

assertThat(antPathMatcher
        .match("https://accounts.google.com/o/oauth2/v2/auth*/**", // (3)
                "https://accounts.google.com/o/oauth2/v2/auth?response_type=code&client_id=x&scope=email%20profile&state=Emrx5OshqhaWyoV82JJ_HZNk28hipx2I3nh6dIFw1Ss%3D&redirect_uri=http://localhost:8080/login/oauth2/code/google"),
        is(true));
```

**(1) 코드**는 처음으로 떠올린 Pattern이다. AntPathMatcher의 pattern이 Query String을 제외하고 판단할 줄 알았던 나의 큰 오산이었다...

**(2) 코드**는 Query String을 pattern에서 포함하여 계산하는 것을 알게되었으니, 0개 이상의 문자를 의미하는 asterisk('\*')를 이용해서 처리하였으나 ***어떤 이유에서인지 테스트가 통과하지 않았다.***

**(3) 코드**는 결국 어떤 문제일지 고민하다가 Query String 중 ***redirect_uri에 포함된 경로***가 문제를 일으키는 것으로 결론지었다. 그렇기에 auth 이하의 모든 문자를 의미하는 **"auth\*"**와 redirect_uri에 포함된 경로까지 고려하는 **"/\*\*"**를 결합하여 pattern을 작성하니 테스트가 통과하였다.

# 💡 결론
***

`Spring Restdocs` 작성을 위해 테스트코드를 작성하는 것이 참 쉽지만은 않다. 내가 걱정했던 부분에서의 오류가 아닌 위와 같은 **예상치못한 곳에서 발생한 것에 대한 처리**가 오래 걸리니 멘탈이 흔들리기도 했다...

그렇지만 RestDocs를 작성하면서 여러 블로그를 탐방하던 중 `가변인자`라는 것에 대해 새로이 알게 되었다.

```java
public void method(String str1, String str2, String str3){}

public void method(String...str){}
```

첫번째 줄의 코드를 두번째 줄과 같이 가변인자를 통해 구현할 수 있는데, 이는 몇개의 인자를 받던지에 상관없이 구성할 수 있다는 장점이 있었다. 실제로 사용해보면 가변인자는 **내부적으로 배열**을 생성해서 사용하는 것을 확인할 수 있었다.