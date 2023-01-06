---
layout: post
title: "[Springboot] unknown collection expression type"
category: tech
tags: troubleshooting spring
---

Springboot에서 JPA Contains 쿼리 메서드를 사용하던 중 `unknown collection expression type` 오류에 대해 포스팅하려 한다.

* unordered toc
{:toc .large-only}

# 👿 Problem
***

Data JPA의 contains 쿼리로 문자열에 포함된 String을 찾아 엔터티들을 반환하게 하려던 중 해당 오류가 발생하였다.

![문제](https://user-images.githubusercontent.com/44282342/205432515-2f35268d-a9a9-4914-b2c3-9cd1bd51a514.PNG)


# 👼 Solution
***

해당 문제는 @Convert 어노테이션으로 DB 내부에는 String으로 저장되고, Entity 객체에는 Collection 타입으로 저장되는데, Contains 쿼리(Like)가 Collection 타입을 조회할 수 없어 발생하는 오류이다.

``` java
    @Convert(converter = StudyTagConverter.class)
    private List<String> tagList;
```

## 해결1. @Formula

![해결](https://user-images.githubusercontent.com/44282342/205432590-94b8af47-5dfc-4c4f-9b40-e7cbdab1d589.png)

`@Formula` 어노테이션을 통해 ***가상의 String 컬럼***을 생성하여, 해당 Column에 DB 내에 저장된 String이 매칭되도록 설정하여 이를 쿼리메서드로 조회한다.
@Formula 는 Native Query를 사용하므로 영향을 받지않는 듯 하다.

## 해결2. Getter, Setter 재정의

![해결2](https://user-images.githubusercontent.com/44282342/205432949-073a0ae7-2361-4b35-a770-674b13f90258.png)

@Convert 어노테이션을 ***사용하지 않고*** String 컬럼의 **Getter를 String to Collection** 으로, **Setter를 Collection to String** 으로 변환하는 로직을 작성하여 사용하는 방법이 있다.
