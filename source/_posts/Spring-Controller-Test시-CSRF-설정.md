---
layout: posts
title: Spring Controller Test시 CSRF 설정
date: 2020-06-09 11:01:52
tags:
    - Spring-Boot
    - Junit
    - Test
---

## 문제

Spring Boot + thymeleaf으로 진행하던 프로젝트를 테스트 하던 중
Ajax로 호출하는 Methoed를 Test 하였더니 아래와 같이 403 error가 return 되면서 Test가 실패했다.

* * *

## 원인

thymeleaf를 사용하면 기본적으로 CSRF 토큰을 넘겨주기 때문에 따로 csrf 토큰을 생성하지 않았으나, ajax에선 직접 csrf 토큰을 생성해서 넘겨줘야 하기 때문에 csrf 토큰을 생성해서 넘겨주도록 되어 있는데 Test 시에는 그렇지 않아서 [403 Forbidden](https://developer.mozilla.org/ko/docs/Web/HTTP/Status/403)가 발생한 것으로 보인다.

![403](/images/20200609/403에러.PNG)

### thymeleaf config.html

```html
<meta id="_csrf" name="_csrf" th:content="${_csrf.token}"/>
<meta id="_csrf_header" name="_csrf_header" th:content="${_csrf.headerName}"/>
```

### common.js

```javascript
const token = $("meta[name='_csrf']").attr("content");
const header = $("meta[name='_csrf_header']").attr("content");
```

* * *
  
## 해결

mockMvc.perform에서 get, post 호출 시 SecurityMockMvcRequestPostProcessors 의 [csrf()](https://docs.spring.io/spring-security/site/docs/4.2.13.RELEASE/apidocs/org/springframework/security/test/web/servlet/request/SecurityMockMvcRequestPostProcessors.html#csrf--)를 [with()](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/web/servlet/request/MockHttpServletRequestBuilder.html#with-org.springframework.test.web.servlet.request.RequestPostProcessor-)로 파라미터에 추가한다.

```java
import static org.springframework.security.test.web.servlet.request.SecurityMockMvcRequestPostProcessors.csrf;

...

//when
ResultActions action = mockMvc.perform(post("/word/add")
        .param("save", "")
        .content(addwordToJson)
        .contentType(MediaType.APPLICATION_JSON)
        .with(csrf())) // 이 부분
        .andDo(print());

...

```

추가 한 후 다시 테스트 해보면 파라미터에 csrf 토큰이 생성되어 정상적으로 동작하는 것을 확인 할 수 있다.

```text
MockHttpServletRequest:
      HTTP Method = POST
      Request URI = /word/add
       Parameters = {save=[], _csrf=[6757e021-c0a3-4390-81d9-9c917d89c8c1]} // 이 부분
          Headers = [Content-Type:"application/json", Content-Length:"59"]
             Body = <no character encoding set>
    Session Attrs = {SPRING_SECURITY_CONTEXT=org.springframework.security.core.context.SecurityContextImpl@83a38fdf: Authentication: org.springframework.security.authentication.UsernamePasswordAuthenticationToken@83a38fdf: Principal: org.springframework.security.core.userdetails.User@31bf3c: Username: jgji; Password: [PROTECTED]; Enabled: true; AccountNonExpired: true; credentialsNonExpired: true; AccountNonLocked: true; Granted Authorities: ROLE_USER; Credentials: [PROTECTED]; Authenticated: true; Details: null; Granted Authorities: ROLE_USER}

...

```

![성공](/images/20200609/success.PNG)

* * *

## 참고 사이트

- [https://stackoverflow.com/questions/25605373/unit-testing-controllers-with-csrf-protection-enabled-in-spring-security](https://stackoverflow.com/questions/25605373/unit-testing-controllers-with-csrf-protection-enabled-in-spring-security)

* * *

## 전체 코드

- [ControllerTest](https://github.com/jungguji/wordbook/blob/master/src/test/java/com/jgji/spring/domain/word/controller/WordControllerTest.java)
- [config.html](https://github.com/jungguji/wordbook/blob/master/src/main/resources/templates/fragments/config.html)
- [common.js](https://github.com/jungguji/wordbook/blob/master/src/main/resources/static/js/common.js)