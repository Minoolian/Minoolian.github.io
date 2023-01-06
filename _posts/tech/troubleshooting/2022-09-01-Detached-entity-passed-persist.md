---
layout: post
title: "[JPA] detached entity passed to persist 오류"
category: tech
tags: troubleshooting spring
---

JPA 연관관계 매핑 중 `detached entity passed to persist` **오류**에 대해 포스팅하려 한다.

* unordered toc
{:toc .large-only}

# 👿 Problem
***

M:N 관계에서 중간테이블을 생성하고 연관관계를 매핑하면서 `@OneToMany`, `@ManyToOne` 어노테이션의 Cascade옵션을 생각한대로 사용하였는데 여러 오류가 발생하였다. 

## CASE 1. `detached entity passed to persist`

```java
public class FavoriteMountain {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long favoriteId;

    @ManyToOne
    @JoinColumn(name = "memberId")
    private Member member;

    @ManyToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "mountainId", nullable = false, referencedColumnName = "mountainName")
    private Mountain mountain; // mountain id를 임의로 설정.

}
```

![image](https://user-images.githubusercontent.com/44282342/187972743-eade28c2-8a3a-4903-902b-945c1eea0d77.png)


Mountain 객체의 **id를 미리 set**해서 FavoriteMountain에 설정하고, Member 객체의 **Cascade 옵션**을 통해 persist한 결과 해당 오류가 발생하였다. 해당 오류는 id값이 미리 설정된 객체가 이미 존재한다(***detaced 상태***)고 판단하여 발생한다.

## CASE 2. `Referential integrity constraint violation`

```java
public class FavoriteMountain {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long favoriteId;

    @ManyToOne
    @JoinColumn(name = "memberId")
    private Member member;

    @ManyToOne
    @JoinColumn(name = "mountainId", nullable = false)
    private Mountain mountain; // mountain id를 임의로 설정.

}
```

![image](https://user-images.githubusercontent.com/44282342/187972920-88ee6520-efa3-468e-a1b7-cd2697fb180d.png)

Mountain 객체의 **id를 미리 set**해서 FavoriteMountain에 설정하고, **Cascade 옵션을 제외**한다면 참조테이블에 존재하지 않는 값을 FK로 설정하기때문에 발생하는 오류이다. (***참조 무결성***)

## CASE 3. `object references an unsaved transient instance`

```java
public class FavoriteMountain {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long favoriteId;

    @ManyToOne
    @JoinColumn(name = "memberId")
    private Member member;

    @ManyToOne
    @JoinColumn(name = "mountainId", nullable = false, referencedColumnName = "mountainName")
    private Mountain mountain; // mountain id를 설정하지 않음.

}
```

![image](https://user-images.githubusercontent.com/44282342/187973180-2c39689b-b469-4b74-a672-8ca351c67fc4.png)

이번에는 Mountain 객체의 **id값을 넣지 않고**, 또한 **Cascade 옵션도 제외**하고 진행하였다. 당연한 결과겠지만 CASE2 결과와 비슷하게 ***참조테이블에 없는 값을 사용하려하기에 발생하는 오류***이다.

# 👼 Solution
***

뾰족한 수가 떠오르진 않았다. 그도 그럴게 참조테이블(Mountain)에 없는 값을 참조하기 위해 값을 **자동으로 추가하는 것이 맞나?** 싶은 생각이 들었다. 만약에 필요하다면 미리 Mountain값을 추가하고 중간테이블에 데이터를 삽입하는 과정으로 진행해야겠다.

# 💡 결론
***

지금 프로젝트의 Mountain 테이블은 Open API를 통해서 자동으로 채워질 예정이기에 해당 부분은 고려하지 않아도 되게 되었다. 하지만 추후에 M:N 연관관계 매핑에서 필요할 부분일 수 있기에 조금 더 자료 조사를 해봐야겠다😎