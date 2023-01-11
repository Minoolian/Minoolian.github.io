---
layout: post
title: "[JPA] 쿼리메서드로 VO 객체 조회?"
category: tech
tags: troubleshooting spring jpa
---

JPA의 쿼리메서드로 VO 객체 조회시 예상치 못했던 결과에 대해 포스팅하려 한다.

* unordered toc
{:toc .large-only}

# 👿 Problem
***

Entity에서 VO 객체를 처음 사용해보는터라 조회할 때, **native 쿼리**나 **JPQL**로 조회해야할 것 같았다. 혹여 객체 자체로 조회를 Data JPA에서 지원하나 싶어 테스팅해보니 실제로 동작하였다.

```java
//file: "StudyService.java"

    public List<StudyArticle> test1() {
        return studyArticleRepository.findByStuMan(new StudyMembers(1, "backend"));
    }
```

![문제1](https://user-images.githubusercontent.com/44282342/213466758-971c1d87-ac36-4be8-9e06-a8f0193079c6.PNG)


# 👼 Solution
***

디버깅을 통해 내부적으로 어떻게 동작하는지 확인해봤지만, Data JPA의 쿼리메서드로 동작하기에 ***찾기가 힘들었다.***

```java
//file: "StudyArticle.java"

    @Embedded
    private StudyMembers stuMan;

```

```java
//file: "StudyMembers.java"

    @Embeddable
    @Getter
    public class StudyMembers {

    private int positionCount;
    private String positionName;

}

```

`@Embedded`와 `@Embeddable` 어노테이션을 통해 StudyArticle Entity의 필드로 포함되었으니, 객체로 조회해도 ***각 필드명을 컬럼에 매칭하여 조회***하는 것으로 보였다.  
다만 VO 객체는 Immutable값을 사용하고, 변경 시 객체를 통채로 바꿔주는만큼 설계면에서 VO 객체 사용이 옳았는지 다시 고려해볼 필요가 있겠다.