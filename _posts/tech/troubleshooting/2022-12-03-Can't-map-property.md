---
layout: post
title: "[Mapstruct] Can't map property"
category: tech
tags: troubleshooting
---

Mapstruct 라이브러리 사용 중 발생한 `Can't map property ~. Consider to declare/implement a mapping method` 오류에 대해 포스팅하려 한다.

* unordered toc
{:toc .large-only}

# 👿 Problem
***

![문제](https://user-images.githubusercontent.com/44282342/205441273-250a134c-6b19-49d3-9fd8-26a37fbc7ec0.PNG)

오류 메시지를 읽어보면, List<String> 타입의 변수를 String 타입의 변수로 변환할 수 없으니 이에 해당하는 Method를 정의하거나 구현된 클래스를 implement 하여 오류를 해결하라고 제시한다.

# 👼 Solution
***

문제에 제시된 것처럼 결국 해당 타입간 변환을 위한 default 메서드를 정의하는 방법을 이용하였다. **다만**, 이전에도 발생했을 때 들었던 ***의문***이 있었다. 만약 변환하는 동일 객체 내에 **또 다른 List<String> to String 로직**이 필요하다면 ***서로 구분***할 수 있어야 하는 것 아닌가에 대한 것이었다.

따라서 이 문제도 같이 해결할 방법을 찾던 중 아래와 같은 좋은 방법이 있었다.

![해결](https://user-images.githubusercontent.com/44282342/205441276-02668c3e-e72e-49c3-84be-4cc4f15b874d.png)

`@Named` 어노테이션으로 구현한 메서드의 식별가능한 이름을 부여하고, 속성 중 `qualifedByName` 속성으로 이름을 지정하면 지정한 이름의 메서드로 변환이 이루어진다.
