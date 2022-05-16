---
layout: post
title: "Spring MVC vs REST API"
# description: >
#   22.04.27 TIL
sitemap: false
---

이번 포스팅은 **Spring MVC vs REST API**의 차이에 대해 알아보려한다.

사실 두 차이를 비교하는 것은 올바르지 않다. 왜냐하면 MVC는 **디자인 패턴** 중 하나이며, Rest API는 네트워크를 통해 통신할 수 있게 하는 **아키텍처 스타일** 이기 때문이다.

그렇다면 포스팅을 통해 확실히 짚고 넘어가도록 하자.

## 디자인 패턴
---

디자인 패턴이란 개발을 할 때, 특정한 문맥에서 **공통적으로 발생하는 문제에 대한 해결책**이다.

이는 내부적으로 제공하는 것이 아닌 프로그래머가 설계할 때 이 문제들을 해결하기 위한 **형식화된 관행**이다.

즉 프로그래머 사이에서 문제를 해결하기 위해 정해진 해결책을 사용하는 **소통의 방법**이라고 생각하면 되겠다.

## MVC 패턴
---

*Model + View + Controller*

![mvc](https://user-images.githubusercontent.com/44282342/168525797-db19a9dc-519e-4cd4-8b74-b7c938f492ce.jpg)

1. Controller를 통해 요청(input)이 들어온다.
2. 해당 요청을 Model에 담는다.
3. 요청을 담은 Model을 보낼 View를 Controller가 선택한다.
4. View단에 Model을 표시한다.

> Controller는 여러 View를 선택할 수 있지만, View를 업데이트하지는 않는다.  
> 현재 사용되는 보편적인 디자인패턴이다.  
> View와 Model 사이의 의존성이 높아 유지보수가 어려울 수 있다.

## MVP 패턴
----

*Mode + View + Presenter*

![mvp](https://user-images.githubusercontent.com/44282342/168526261-d1270211-b7b4-425e-a882-f98a311c71a8.jpg)

1. View를 통해 요청이 들어온다.
2. View에서 Presenter에 Data를 요청한다.
3. 받은 요청을 Presenter가 Model에게 전달한다.
4. Model에서 해당 요청을 담아 Presenter에 응답한다.
5. 해당 응답을 View에 넘겨 화면에 표시한다.

> Model과 View를 연결하는 다리 역할을 한다.  
> View와 Model간 의존성이 없어 MVC패턴의 단점을 해결하였다.  
> 다만, View와 Presenter 사이 의존성이 강해진다.

## MVVM 패턴
---

*Model + View + View Model*

![mvvm](https://user-images.githubusercontent.com/44282342/168526712-da790e4e-6258-43d4-ab4d-b1487eb892c6.jpg)

1. View를 통해 요청이 들어온다.
2. Command 패턴으로 View Model에 요청을 전달한다.
3. View Model은 받은 요청을 Model에 전달한다.
4. Modle에서 해당 요청을 담아 View Model에 응답한다.
5. View Model에서 응답받은 Data를 가공하여 저장한다.
6. View에서 View Model이 가진 Data를 Binding하여 화면에 표시한다.

> Command, Data Binding 패턴을 사용하여 구현하였다.  
> View와 Model, View와 View Model 사이의 의존성 모두 없다. (독립적이다)  
> 다만, View Model의 설계가 어렵다.


## API (Application Programming Interface)
---

정의된 프로토콜을 이용하여 양측간 상호작용을 할 수 있도록 약속된 시스템을 의미한다.

쉬운 예로, C언어에서 텍스트를 출력하는 printf API를 사용하여 표준 출력을 작성한다. 이것은 각 OS에서 모두 동작할 수 있도록 **C언어 API가 보장**한다.  
printf는 API를 기반으로 설계된 문법이며 이것들이 모이면 '라이브러리'가 되는 것이다.

즉 **API가 없으면** 프로그래머가 Low Level에서 원하는 동작을 하기 위해 메모리를 건드려야하는 위험한 작업을 해야한다.

위처럼 프로그램과 OS 간의 상호작용을 위한 프로토콜이 있는 반면, 기업 서버에서 개발자가 필요한 부분만 상호작용할 수 있게 만든 API(ex. 카카오맵 API)도 있다.

## REST API
---

***REST(Representational State Transfer) API는 네트워크를 통해 컴퓨터간 통신할 수 있게 하는 아키텍처 스타일***이다.  
**URI**와 **HTTP 프로토콜**을 기반으로 **JSON 데이터**를 주고 받는다.

*REST 방식*은 **Web에 최적화**되어 있고, *JSON 데이터*를 이용하므로 **브라우저간 호환성**이 좋다.

비슷한 방식으로 REST API의 문제를 해결하기 위해 나온 **GraphQL**과 많은 리소스, 엄격한 보안 등이 필요하다면 **SOAP API**가 있다. 

* REST 구성
  * 자원(Resource): URL
  * 행위(Verb): HTTP Method
  * 표현(Represetation): 적절한 응답

## REST의 디자인 가이드
---

1. URI는 자원의 정보를 표현해야 한다.
2. 자원의 행위는 HTTP Method로 표현해야 한다.

```plain
GET /post/1
POST /post/1
```

|Method|역할|
|:--|:--:|
|GET|리소스를 생성한다.|
|POST|리소스를 조회한다.|
|PUT|리소스를 수정한다.|
|DELETE|리소스를 삭제한다.|

> 자원을 표시하는 데 중점을 두고 행위의 표현이 URI내에 들어가서는 안된다.

### REST 디자인 시 주의할 점
---

1. '/' 는 계층 관계를 나타낸다.

```plain
/subway/line1
```

2. URL 마지막 문자에 '/'를 포함하지 않는다.
3. '-'은 URL 가독성을 높이는데 사용한다.
4. '_'은 URL에 사용하지 않는다.
5. URL 경로에는 소문자를 사용한다.
  URL 경로에 대문자가 있다면, 대소문자 구별에 따라 다른 리소스로 구분될 수 있기 때문이다.
6. 파일 확장자는 URL에 포함하지 않는다.
  Header에 따로 포함시킨다.

### Resource 간의 관계를 표현
---

```plain
/리소스명/리소스 ID/관계있는 리소스명

GET /users/{userid}/devices ('has'의 관계를 표현할 때)
GET /users/{userid}/likes/devices (관계명이 애매하거나 구체적인 표현이 필요할 때)
```

### 자원을 표현하는 Collection과 Document
---

```plain
/posts/1/comments/2
```

posts, comments 컬렉션(다수)이 각각 Id에 해당하는 리소스를 의미한다.  
즉, Collection과 Document를 사용할 때  복수/단수로 정의하여 직관적인 REST API로 표현할 수 있다.

## HTTP Status
---

마지막으로 서버에서 적절한 응답을 주는 것 까지 포함되어야 **RESTful**하게 설계하는 것이다.  
HTTP Status만으로 효과적으로 응답할 수 있기에 중요하다.

|Status|Mean|
|:--:|:--|
|200|클라이언트의 요청을 정상적으로 수행|
|201|클라이언트의 리소스 생성 요청에 대한 성공 (POST)|
|400|클라이언트의 요청이 부적절|
|401|인증되지 않은 클라이언트가 보호된 리소스 요청|
|403|클라이언트가 응답하고 싶지 않은 리소스를 요청 <br> 400 또는 404 사용을 권고(403이 리소스가 존재한다는 뜻이기에)|
|405|클라이언트가 요청한 리소스에서 사용 불가능한 Method를 이용|
|301|클라이언트가 요청한 리소스에 대한 URI가 변경(Redirection)|
|500|서버에 문제가 있을 경우|

## 결론
---

즉, 내가 헷갈려 온 두 개념은 상반된 개념이 아닌, ***'MVC 기반의 REST API를 설계한다'*** 라는 의미로 이해를 했어야 하는 것이다😅

이처럼 아직 겉으로만 알고있는 미흡한 개념들이 많을 것이다. 하나하나 정리하여 머리속에 꽉꽉 채우도록 하자❗

> 이미지 참고: https://swimjiy.github.io/2019-05-28-web-mvc-mvp-mvvm