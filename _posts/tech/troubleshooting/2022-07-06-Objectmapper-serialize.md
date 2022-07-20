---
layout: post
title: "[Spring] Objectmapper로 LocalDateTime Serialize시 오류"
category: tech
tags: troubleshooting
---

**Objectmapper로 LocalDateTime을 Serialize할 때 발생한 오류**에 대해 포스팅하려 한다.

* unordered toc
{:toc .large-only}

# 👿 Problem
***

> Java 8 date/time type `java.time.LocalDateTime` not supported by default: add Module "com.fasterxml.jackson.datatype:jackson-datatype-jsr310"...

예외처리를 위해 **ObjectMapper**로 **Response Body**에 `LocalDateTime` 타입을 전송하려던 중 예상치 못한 Error에 봉착하였다....

# 👼 Solution
***

```java
ObjectMapper om = new ObjectMapper();
        // LocalDateTime 자료형에 대해 Objectmapper serialize 오류를 발생한다.
        om.registerModule(new JavaTimeModule()).disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
```

**Java 8에서 등장**한 `LocalDateTime`을 Serializae 하기 위해서는 위와 같이 추가적인 설정이 필요하다고 한다.

**배열형식**으로 LocalDateTime이 찍히는 데 이를 ***String*** 형태로 표현하기 위해 `SerializationFeature.WRITE_DATES_AS_TIMESTAMPS` 속성을 사용한다.
