---
layout: posts
title: Spring Boot + Vue CORS 설정
date: 2021-03-28 18:40:28
tags:
    - Spring Boot
    - Vue
    - CORS
---

## 서론

Front를 Vue.js, Back을 Spring Boot로 만든 토이 프로젝트에서 CORS로 인해 통신이 되지 않는 오류가 발생하여 문제를 해결한 방법을 작성 해둔다.

* * *

## 문제 상황

화면에서 체크박스를 클릭하면 서버로 requert를 보내고 서버에서 그에 맞는 response를 주는 방식에 간단한 프로젝트 인데 클릭 시 아래 이미지 처럼 'Network Error'라는 alert를 발생시키고 통신이 되지 않는 문제가 발생 하였고, 개발자 도구로 콘솔을 확인해보니 아래 이미지 처럼 'CORS Preflight Did Not Succeed'과

> 교차 출처 요청 차단: 동일 출처 정책으로 인해 http://localhost:9312/frame에 있는 원격 자원을 차단하였습니다. (원인: CORS 사전 점검 응답 실패).

라는 메시지가 출력된 것을 확인했다.

{% asset_img error.PNG Spring Boot + Vue CORS 설정 'CORS Error 에러 발생' 'CORS Error 에러 발생' %}

* * *

## 해결 방법

여러 가지 방법이 있겠지만 필자는 WebMvcConfigurer를 상속받아 설정을 추가해주는 방식으로 처리했다.

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("http://localhost:8080")
                .allowCredentials(true);
    }
}

```

1. 위 코드 처럼 WebMvcConfigurer 를 상속받는 WebConfig class를 작성한다.
2. 그 중 [addCorsMappings()](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/config/annotation/WebMvcConfigurer.html#addCorsMappings-org.springframework.web.servlet.config.annotation.CorsRegistry-) 메서드를 override 한다.
3. CorsRegistry 클래스의 [addMapping()](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/config/annotation/CorsRegistry.html#addMapping-java.lang.String-) 메서드를 통해 CORS 요청 처리를 활성화할 URL를 지정하는데 이 때 "/**" 같은 Ant 스타일 패턴이나 정확한 경로(ex /admin)를 지정하는 것도 가능하다.
4. 그 후 [allowedOrigins()](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/config/annotation/CorsRegistration.html#allowedOrigins-java.lang.String...-) 메서드에서 CORS 요청을 허용 할 URL를 지정한다.
5. 참고로 allowCredentials 설정을 true로 줬는데 이렇게 Access-Control-Allow-Credentials를 true로 할 경우 allowedOrigins()에서 "*"로 해서 모든 요청에 대해 CORS를 허용 할 수 없다.

이 방법 외엔 [@Crossorigin](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/CrossOrigin.html)을 이용해 개별 클래스 혹은 메서드에 CORS 요청 인증을 응답하도록 설정 하는 것도 가능하다.

### 정상 동작한 모습

{% asset_img 정상.PNG Spring Boot + Vue CORS 설정 "정상적으로 통신하여 체크된 상황에 대한 코드를 응답한 모습" "정상적으로 통신하여 체크된 상황에 대한 코드를 응답한 모습" %}

* * *

## 여담

이전에 개발 할 때는 설정하지 않아도 잘 됐던거 같은데 오랜만에 프로젝트를 클론받아 실행하니 동작하지 않아 당황했다... 왜 그럴까?

* * *

## 참고 사이트

- [https://velog.io/@logqwerty/CORS](https://velog.io/@logqwerty/CORS)
- [https://developer.mozilla.org/ko/docs/Web/HTTP/CORS](https://developer.mozilla.org/ko/docs/Web/HTTP/CORS)
- [https://dev-pengun.tistory.com/entry/Spring-Boot-CORS-%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0](https://dev-pengun.tistory.com/entry/Spring-Boot-CORS-%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0)
- [https://reikop.com/springboot-vue-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-%EC%84%B8%ED%8C%85%ED%95%98%EA%B8%B02/](https://reikop.com/springboot-vue-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-%EC%84%B8%ED%8C%85%ED%95%98%EA%B8%B02/)
