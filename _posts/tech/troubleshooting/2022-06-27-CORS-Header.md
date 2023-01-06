---
layout: post
title: "[JWT] CORS 해결 후 프론트단 토큰 저장 오류"
category: tech
tags: troubleshooting spring
---

CoCo 프로젝트 중 **CORS 해결 후 프론트단의 JWT 저장 문제**에 대해 포스팅하려 한다.

* unordered toc
{:toc .large-only}

# 👿 Problem
***

![token trouble](https://user-images.githubusercontent.com/44282342/175888156-23f4b9cf-7cfd-4d02-a3d6-57a7cad8ef4f.PNG)

리액트와 스프링부트 통합으로 빌드를 진행했을 때는 `SOP정책에 위반되지 않아` CORS에 대한 처리가 필요하지 않았다. 하지만 프론트 팀원들이 **VSCode에서 코드를 수정사항을 바로 반영하여 개발**을 하고싶다고 요청하여 별개로 빌드하여 작동해보았다.

아니나 다를까 CORS가 발생했지만, 서버단에서 처리가 되어있었기에 API 요청은 정상적으로 진행되었다. 하지만! preflight 요청과 함께 `Authorization Header`가 정상적으로 수신되지 않는 것을 확인하였다.... 즉, JWT 토큰 인증에 문제가 발생한 것이다.

# 👼 Solution
***

```java
//file: "CorsConfig.java"
@Configuration
public class CorsConfig {

    @Bean
    public CorsFilter corsFilter() {
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        CorsConfiguration config = new CorsConfiguration();
        config.setAllowCredentials(true);
        config.addAllowedOrigin("http://localhost:3000");
        config.addAllowedHeader("*");
        config.addAllowedMethod("*");
        config.addExposedHeader("Authorization"); // CORS로 인해 프론트단에서 인식하지 못하는 Authrization 헤더를 노출

        source.registerCorsConfiguration("/**", config);
        return new CorsFilter(source);
    }
}
```

생각보다 해결은 간단했다. `addExposedHeader`에 `Authorization을 추가`하여 클라이언트에서 해당 헤더에 접근할 수 있게 설정하면 정상적으로 Token을 수신하여 저장할 수 있다.

# 💡 결론
***

처음 도입해보는 JWT를 구현하면서 **시행착오들**이 많다. accessToken 뿐 아니라 현재 refreshToken을 추가하려고 하는데, 상당히 애를 먹고있다😥 발생하는 문제들에 TroubleShooting을 기록하여 다시 발생했을때 빠르게 대처할 수 있는 능력을 길러야겠다.