---
layout: post
title: "[Gradle Build] Intellij 안의 Gradle Build 오류"
category: tech
tags: troubleshooting spring
---

Intellij 툴 내에 있는 Gradle Build 시 발생하는 오류에 대해 포스팅하려 한다.

* unordered toc
{:toc .large-only}

# 👿 Problem
***

![gradle 문제](https://user-images.githubusercontent.com/44282342/192290651-414c9b17-02b6-4fee-8a57-76dd5fe7bf4c.PNG)

![gradle 문제2](https://user-images.githubusercontent.com/44282342/192290643-a941097b-891e-4699-b688-6733dc7dc014.PNG)

Intellij 내에 있는 Gradle Build를 사용할 일이 많지는 않았다. `./gradlew build` 명령을 통해서 빌드를 진행했었고, 이를 통해 jar 파일로 서버를 구동했기 때문이다. 하지만 API 명세를 위해 **Restdocs를 사용**하면서 보다 가벼운 Intellij 내의 build를 하려하니 오류가 발생하였다...

# 👼 Solution
***

이유는 생각보다 간단했지만 찾는데 시간은 정말 오래걸렸다. 해당 오류를 보면 프로젝트 경로중에 `한글명`이 포함된 것을 볼 수 있다. 즉, 빌드를 하면서 한글 파싱에 대한 오류가 있는 것으로 보인다...

# 💡 결론
***

> 한글경로 사용은 가능하면 **지양**하자.