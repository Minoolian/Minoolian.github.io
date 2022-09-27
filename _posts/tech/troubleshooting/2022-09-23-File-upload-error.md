---
layout: post
title: "[SpringBoot] DTO + MultipartFile 같이 사용시 오류"
category: tech
tags: troubleshooting
---

SpringBoot Controller의 핸들러메서드에서 게시글에 `필요한 DTO 데이터`와 이미지 저장을 위한 `MultipartFile`을 함께 사용시 발생한 오류에 대해 포스팅하려 한다.

* unordered toc
{:toc .large-only}

# 👿 Problem
***

## 문제1. 기존 @RequestBody + @RequestParam

```java
    @PostMapping
    public ResponseEntity<ProductResponseDto.GetDetail> createProduct(
            @RequestBody ProductRequestDto.Post postDto,
            @RequestParam("file") List<MultipartFile> file) throws IOException {

    ...

    }
```

REST API에서 JSON 타입과 multipart/form-data 타입을 함께 사용해본적이 없어 우선 기존에 사용했던대로 적용해보았더니 오류가 발생했다.

```
2022-09-23 02:11:47.060  WARN 5624 --- [nio-8080-exec-2] .w.s.m.s.DefaultHandlerExceptionResolver :
 Resolved [org.springframework.web.HttpMediaTypeNotSupportedException: Content type 'multipart/form-data;
 boundary=--------------------------316474348119422502495503;charset=UTF-8' not supported]
```

JSON 타입을 MultipartFile로 **역직렬화를 할 수 없어** 발생하는 오류로 보인다.

## 문제2. @RequestPart + @RequestPart

```java
    @PostMapping
    public ResponseEntity<ProductResponseDto.GetDetail> createProduct(
            @RequestPart ProductRequestDto.Post postDto,
            @RequestPart("file") List<MultipartFile> file) throws IOException {

    ...

    }
```

**@RequestPart** 어노테이션은 MultipartFile의 경우 `MultiPartResolver`를 통해 역직렬화가 가능하고, 없는 경우는 @RequestBody와 마찬가지로 `HttpMessageConverter`를 통해 JSON 타입도 역직렬화가 가능하다고 한다.

```
2022-09-23 02:28:27.868  WARN 14252 --- [nio-8080-exec-6] .w.s.m.s.DefaultHandlerExceptionResolver :
 Resolved [org.springframework.web.HttpMediaTypeNotSupportedException: Content type 'application/octet-stream' not supported]
```

??!! 예상치 못했던 오류가 발생했다. 왠지 MultipartFile은 잘 넘어온 것으로 보였고, **JSON 타입의 데이터에 문제**가 생긴 것으로 생각했다. 왜냐하면 API를 테스트할 때 Body에 JSON 데이터를 보내는 것이 일반적이었는데, form-data 타입은 parameter에 담아 보내야하므로 차이점이 있었기 때문이다.

# 👼 Solution
***

![해결3](https://user-images.githubusercontent.com/44282342/192285742-256a445c-47f6-44ac-b089-db3ba1fd30a7.PNG)

혹시가 역시였다. form-data 타입의 key-value 값으로 보낼때에는 **Content-type을 application/json 타입**으로 명시해주어야 객체로 역직렬화가 정상적으로 이루어지는 것이다.


# 💡 결론
***

과거에 MVC 패턴으로 개발했을 때는 View에서 `@ModelAttribute` 어노테이션을 통해 `form-data`를 받았기 때문에 문제가 없었지만, REST API로 개발할때 보통 `JSON` 타입의 데이터를 받다보니 많이 헤맸다. 해결한 문제를 바탕으로 S3 업로드와 에디터에서의 이미지 삽입/삭제를 수월하게 진행할 수 있을 것 같다😎