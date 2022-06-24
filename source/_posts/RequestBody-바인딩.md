---
layout: posts
title: RequestBody 바인딩
date: 2022-03-31 22:44:43
tags:
    - spring
    - MessageConverter
---

## 서론

이전에 작성한 글 중에 [@Requestbody Annotation 사용 시 기본 생성자 필요](https://jungguji.github.io/2021/02/05/Requestbody-Annotation-%EC%82%AC%EC%9A%A9-%EC%8B%9C-%EA%B8%B0%EB%B3%B8-%EC%83%9D%EC%84%B1%EC%9E%90-%ED%95%84%EC%9A%94/) 라는
글을 올린 적이 있는데 글의 마지막에 보면

{% blockquote JG.JI, @Requestbody Annotation 사용 시 기본 생성자 필요 %}
@Requestbody가 jackson 라이브러리를 이용해 매핑하기 때문이다.
{% endblockquote %}

<br />

라는 결론으로 글을 마무리 하였는데, [김영한님의 강의](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-1/dashboard)에서 해당 내용과 관련해서 자세하게 알려주시는 부분이 있어, 해당 내용을 정리해본다.

* * *

## 본론

우선 Spring에서는 Controller에서 @RequestBody로 넘어오는 파라미터들에 대해서 내부적으로 ObjectMapper를 이용해 JSON 데이터를 우리가 만든 DTO 객체로 변환시켜주고,
@ResponseBody인 경우엔, DTO 객체를 JSON 데이터로 변환해서 클라이언트에 넘겨준다.

이렇게해서 해소 될 궁금증이었으면 이 글을 읽으러 오지 않았을테니, 해당 내용에 대하여 코드 레벨에서 자세히 알아보기로 한다.

우선, Spring MVC의 동작 순서를 간략하게 서술하자면(구글에 검색하면 훌륭한 분들이 작성하신 훌륭한 글들이 많으므로)

1. 클라이언트에서 요청이 넘어오면 FrontController 역할을 수행하는 DispatcherServlet에서 먼저 확인해서,
2. 핸들러 매핑 리스트 중 해당 요청을 처리 할 수 있는 핸들러를 가져오고,
3. 핸들러 어댑터 리스트 중에서 해당 핸들러를 처리 할 수 있는 핸들러 어댑터를 가져와서,
4. 해당 어댑터에서 우리가 만든 컨트롤러에 호출해서 비즈니스 로직을 처리 한 후,
5. 리턴되는 데이터에 맞게 뷰 리졸버나 메시지컨버터가 동작하고,
6. 알맞게 클라이언트로 반환된다.

여기서, @RequestMapping에 대한 request는 [RequestMappingHandlerAdapter](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/mvc/method/annotation/RequestMappingHandlerAdapter.html)에서 처리하는데
이 RequestMappingHandlerAdapter는
    1. byte[]를 처리하기 위한 ByteArrayHttpMessageConverter
    2. String을 처리하기 위한 StringHttpMessageConverter
    3. json을 처리하기 위한 [MappingJackson2HttpMessageConverter](https://docs.spring.io/spring-framework/docs/current/javadoc-api/index.html?org/springframework/http/converter/json/MappingJackson2HttpMessageConverter.html)
등을 가지고 있다.

### RequestMappingHandlerAdapter에 설정된 MessageConverter들

{% asset_img 1_RequestMappingHandlerAdapter_messageConverters.JPG RequestBody 바인딩 "RequestMappingHandlerAdapter에 설정된 MessageConverter들" "'RequestMappingHandlerAdapter에 설정된 MessageConverter들'" %}

<br />

그리고 자신이 가지고 있는 ArgumentResolver들 중
@RequestBody형식으로 들어오는 파라미터를 처리하기 위해 RequestResponseBodyMethodProcessor 라는 ArgumentResolver를 호출한다.

### RequestMappingHandlerAdapter에 설정된 ArgumentResolver들

{% asset_img 2_RequestMappingHandlerAdapter_ArgumentResolvers.JPG RequestBody 바인딩 %}

<br />

이 RequestResponseBodyMethodProcessor의 readWithMessageConverters 메서드에서 메시지 컨버터 리스트를 loop 돌면서
대상 클래스 타입과 미디어 타입 등을 체크해서 알맞은 메시지 컨버터를 호출해서 사용하는데
이 때 호출되는 MessageConverter가 MappingJackson2HttpMessageConverter라는 MessageConverter이다.

### 요청을 처리 할 수 있는 MessageConverter 호출

{% asset_img 3_AbstractMessageConverterMethodArgumentResolver_readWithMessageConverters.JPG RequestBody 바인딩 %}

1. MessageConverter의 canRead() 메서드로 해당 요청을 처리 할 수 있는 컨버터 인지 확인.
2. 처리 할 수 있으면 read() 메서드로 메시지에서 객체를 읽고 반환한다.

<br />

MappingJackson2HttpMessageConverter에서는 최종적으로 readJavaType 메서드가 호출되는데 이 때 objectMapper를 사용해서 json 데이터를 DTO 객체로 변환하는 것을 볼 수 있다.

### ObjectMapper를 이용해서 JSON데이터를 변환하는 부분

{% asset_img 4_AbstractJackson2HttpMessageConverter_readJavaType.JPG RequestBody 바인딩 %}

* * *

## 결론

보이지 않는 곳에서 이렇게 많은 부분을 스프링에서 처리해주고 있었기 때문에 우리는 Controller에서 편하게 파라미터를 사용 할 수 있고, JSON 데이터도 바로 객체로 받을 수 있는 것이었다.
이런 내용을 알아도, 몰라도 결국 같은 방식으로 개발 하겠지만... 언젠가 도움이 되겠거니 생각하고 지금은 궁금증을 해결 했다는 것에 대해 만족한다.
이 글을 읽는 다른 분들도 필자와 같은 궁금증이 있었다면 조금이나마 해소가 됐길 바라며..

* * *

## 참고 사이트

- [https://velog.io/@conatuseus/RequestBody%EC%97%90-%EA%B8%B0%EB%B3%B8-%EC%83%9D%EC%84%B1%EC%9E%90%EB%8A%94-%EC%99%9C-%ED%95%84%EC%9A%94%ED%95%9C%EA%B0%8](https://velog.io/@conatuseus/RequestBody%EC%97%90-%EA%B8%B0%EB%B3%B8-%EC%83%9D%EC%84%B1%EC%9E%90%EB%8A%94-%EC%99%9C-%ED%95%84%EC%9A%94%ED%95%9C%EA%B0%8)
- [https://csy7792.tistory.com/317](https://csy7792.tistory.com/317)
- [김영한 - 스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-1)