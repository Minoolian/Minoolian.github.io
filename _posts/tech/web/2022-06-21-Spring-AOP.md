---
layout: post
title: "Spring AOP"
category: tech
tags: web spring
image:
  path:   https://user-images.githubusercontent.com/44282342/174726342-1dd83191-8f51-42cc-8c1a-76df9f3c1be2.png
---

* unordered toc
{:toc .large-only}

이번 포스팅은 **Spring AOP**에 대해 알아보려한다.

## AOP (Aspect Oriented Programming)
***

`핵심 관심사(핵심기능)`와 `횡단 관심사(부가기능)`가 공존하면서 메서드의 복잡도가 높아지고, 코드가 지저분해지게 되었다. 따라서 OOP만으로는 이 문제를 해결할 수 없기에 **AOP가 등장**하였다.

스프링에서 AspectJ를 사용하기 위해 최상위 패키지 클래스에 `@EnableAspectJAutoProxy` 어노테이션을 추가해주어야 한다.

```java
@EnableAspectJAutoProxy
@SpringBootApplication
public class DemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(CmarketApplication.class, args);
	}
}
```

## AOP 용어 및 개념
***

![Advice와 JoinPint의 관계](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Faa82f0ab-a955-40e4-a15a-4d77661833c2%2FUntitled.png?table=block&id=0a3c2d8f-bd8e-46cd-8d8d-00936442fbd3&spaceId=f6562176-b9b7-40a3-8387-ce49c6f98fe8&width=2000&userId=7ff29728-065b-4cd4-bf01-375a94766527&cache=v2)

### Aspect

* Concern을 묶어서 모듈화한 것. 하나의 모듈. Advice와 PointCut이 들어간다.

### JoinPoint

* 클래스 초기화, 객체 인스턴스화, 메소드 호출 등의 특정 포인트를 의미한다. 다만 스프링 AOP에서는 **프록시 방식**을 사용하므로 `메서드 실행 시점`으로 제한된다.
* 조인포인트에 관심코드로 새로운 동작을 추가할 수 있다.

### Advice

* JoinPoint에서 수행되는 코드이다.
* Aspect를 핵심코드에 언제 적용할지 선언한다.

### Pointcut

* Advice가 적용될 JoinPoint 위치를 결정한다.
* AspectJ 표현식을 사용하며, 위에 언급했듯 스프링 AOP에서는 메소드 실행지점만 가능하다.

### Target

* **Advice를 받는** `객체`, 즉 핵심 기능을 담고 있는 객체이다.

## Advice의 종류
***

|종류|설명|
|:--|:--|
|Before|조인 포인트 실행 이전|
|After returning|조인 포인트 정상 완료후|
|After throwing|메서드에서 예외를 던지는 경우|
|After|정상과 예외에 상관없이 실행(finally)|
|**Around**|메서드 호출 전후에 수행|
{:.stretch-table}

모두 메서드의 인자로 `JoinPoint`를 받지만 `@Around` 어노테이션만 `ProceedingJoinPoint`를 인자로 받는다.

`@Around` 어노테이션으로 모두 처리 가능하지만, 간결하게 **필요한 어노테이션**을 사용하는 것이 좋은 설계라할 수 있다.

> Advice 순서는 보장되지 않으므로 클래스 레벨에 적용하는 @Order 어노테이션에 대해 알아보면 좋을 것 같다.

## Pointcut 표현식
***

`execution([접근제한자 패턴] 리턴값의 타입 [패키지, 클래스명] 메서드명(파라미터타입) [throws 예외패턴])`

```java
exectuion(* com.example.demo.*.*(..))
```

## JoinPoint
***

스프링 AOP의 JoinPoint는 **메서드 실행 시점**이며, **스프링 Bean**에만 적용할 수 있다.

### JoinPoint 인터페이스

|메서드|설명|
|:--|:--|
|getArgs()|JoinPoint에 전달된 인자 반환|
|getThis()|AOP 프록시 객체 반환|
|getTarget()|AOP 적용된 객체 반환|
|getSignature()|Advice가 적용되는 메서드에 대한 설명을 반환|
{:.stretch-table}

`@Around` 어노테이션에서 사용하는 `ProceedingJoinPoint 인터페이스`는 다음 Adivce를 호출하는 `proceed()` 메서드가 있다.

## 결론
***

AOP를 통해 간단한 메서드 실행시간 측정정도는 수행해보았지만 실질적으로 사용될 **트랜잭션이나 로깅**부분은 내 손으로 직접 짜보질 않았다😥 아마 지금 진행중인 CoCo 프로젝트에서 분명히 필요한 부분이거니와 코드스테이츠 커리큘럼에도 포함되어있으니 `내 손으로 구현할 날이 올 것이라 기대한다😉`

또한 프로젝트에서 Controller단에 발생하는 예외를 `Exception Handling`할 예정이었는데 마침 AOP에 대해 학습하여 수월하게 진행할 수 있을 것이다.