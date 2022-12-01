---
layout: post
title: "hashCode & equals?"
category: tech
tags: web
image:
  path:   https://user-images.githubusercontent.com/44282342/205482196-c8fbae08-98da-44ce-822e-7b7d99b4c315.png
---

* unordered toc
{:toc .large-only}

이번 포스팅은 **hashCode() 와 eqauls()**에 대해 알아보려한다.

## 필요성
***

PS를 하면서 풀이방법을 설계하던 중, 객체 내 **인스턴스변수 중 특정 변수가 같으면** 두 객체가 **같음**을 정의하고 싶었다. 문득 JPA에서 Collection 자료형을 사용할 때 Collection내에 원하는 객체가 제대로 찾아지지 않던 경우가 생각났다. 그 당시에는 결국 Collection을 순회하며 찾았던 기억이 떠오르는데, 이 문제와 흡사한 경우인것 같아 방법을 찾아보게 되었다.

## equals()
***

두 객체가 같은지 검사하는 메서드로 기본적으론 주소값을 비교하게 구현되어있다.

```java
//file: "Object.java"
public boolean equals(Object obj) {
        return (this == obj);
}
```

하지만 언급했듯, 프로그래밍을 하다보면 특정 값으로 비교하는 것과 같은 **다른 기준에 의해 객체가 동일하다 판단**을 해야할 경우가 있다. 이럴 경우 `eqauls()` 메서드를 오버라이딩하여 재정의하여 기준을 다시 세워준다.

```java
static class Node {
        int x;
        int y;
        int dir;

    ...

        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (o == null || getClass() != o.getClass()) return false;
            Node node = (Node) o;
            return x == node.x && y == node.y;
        }
}
```

위 코드는 PS를 하면서 풀었던 구현 문제중 좌표를 담는 Node 객체중 일부이다. 해당 클래스의 **x, y의 값이 같으면** 객체가 동등하다 판단하였기에 equals() 메서드를 위와 같이 재정의하였다.


## hashCode()
***

Hash 값을 사용하는 Collection 타입(HashSet, HashMap, HashTable)을 사용할 때, 해당 자료의 저장 위치를 정하기 위해 사용된다.

```java

static class Node {
        int x;
        int y;
        int dir;

    ...

        @Override
        public int hashCode() {
            return Objects.hash(x, y);
        }
}
```

알다시피 HashSet과 같은 자료구조에서는 **같은 값의 중복이 일어나면 안되기에** 같은 두개의 객체는 자료구조에서 ***같은 위치에 저장***되어 하나로 취급되어야 한다. 이럴 때 필요한 것이 hashCode() 메서드이다.


## 결론
***

이 방법을 통해서 PS 풀이를 할 때 두 좌표객체의 동일성을 if문과 함께 검증할 수 있었다. 또한 추후 JPA Entity 객체에서 맞닥뜨렸던 문제를 다시 한번 접하게 된다면 지금 알게된 새로운 방법으로 접근하여 해결할 수 있을 것이다.

여러 자료를 검색하다보니 ***Hash를 이용한 자료구조를 사용하지 않으면*** `hashCode()` 메서드는 재정의하지 않아도 무방하지 않은가? 라는 의문에 대한 글을 많이 접한것 같다. 필자들마다 조금은 의견이 상이하지만 결국 Hash 자료구조를 사용할 일말의 가능성이 있기에 `equals()` 메서드를 재정의한다면 동시에 `hashCode()` 메서드도 **함께 재정의하는 것이 맞다**고 본다.