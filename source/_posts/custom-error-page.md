---
layout: posts
title: 'Spring boot + thymeleaf custom error page 적용'
date: 2020-04-15 17:44:24
tags:
    - Spring-Boot
    - thymeleaf
---
# 서론

기본적으로 was나 spring boot에서 error page를 제공해 주고 있으나

보기 이쁘지도 않을뿐더러 stacktrace 가 그대로 노출되기 때문에 보안상으로도 좋을 것이 없다.

* * *
# 본론

하여 보통 에러 페이지를 따로 개발하여 보여주게 되는데 thymeleaf과 spring boot을 사용할 경우

[https://www.thymeleaf.org/doc/articles/springsecurity.html](https://www.thymeleaf.org/doc/articles/springsecurity.html)

위 주소의 tymeleaf 공식 홈페이지에서 제시한 방법처럼 적용하면 된다.

이 경우 spring security가 전혀 관여하지 않기 때문에 ExceptionHandler를 추가할 것을 권하고 있다.

나는 에러 유형에 따라 이미지 파일을 보여주도록 custom error page 기능을 아래 처럼 적용했다.

## Controller

```java
@Controller
public class CustomErrorController implements ErrorController {

    private static String IMAGE_PATH = "images/error/";
    
    @ExceptionHandler(Throwable.class)
    @GetMapping("/error")
    public String handleError(HttpServletRequest request, Model model) {
        Object status = request.getAttribute(RequestDispatcher.ERROR_STATUS_CODE);
        String statusMsg = status.toString();
        
        HttpStatus httpStatus = HttpStatus.valueOf(Integer.valueOf(statusMsg));
        
        model.addAttribute("msg", statusMsg + " " + httpStatus.getReasonPhrase());
        model.addAttribute("src", IMAGE_PATH + statusMsg + ".jpg");
        
        return "thymeleaf/error";
    }
    
    @Override
    public String getErrorPath() {
        // TODO Auto-generated method stub
        return "/error";
    }
}
```

## View

```html

<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
    <head>
        <title>Error page</title>
        <meta charset="utf-8" />
        <link rel="stylesheet" href="css/error.css" th:href="@{/css/error.css}" />
    </head>
<body>
    <img th:alt="${msg}" th:src="${src}">
</body>
</html>
```

### thymeleaf를 사용하지 않는 경우

thymeleaf를 사용하지 않을 경우엔 resource 폴더 하위에 static 폴더에 error 페이지를 만들어놓고 CustomController를 아래처럼 [IllegalStateException](https://docs.oracle.com/javase/8/docs/api/java/lang/IllegalStateException.html) 을 throw 하기만 하면 된다.

## Controller

```java

public class CustomErrorController implements ErrorController {

    private String PATH = "/error";
    
    @GetMapping("/error")
    public String errorpage(){
        throw new IllegalStateException("Error");
    }
    
    @Override
    public String getErrorPath() {
        // TODO Auto-generated method stub
        return PATH;
    }
}
```

* * *
# 결론

## 적용 결과

![결과](/images/error-page/error.PNG)

나는 [https://http.cat/](https://http.cat/) 라는 http status code마다 위처럼 고양이 사진을 보여주는 사이트에서 이미지 파일을 가져와 적용하였다.

그 외에 다른 사이트들에선 어떻게 에러 페이지를 보여주는지 확인해보고 각자 에러 페이지를 작성하면 좋을 것이다.

[[번역] '404 에러페이지' 24가지 분석 -1](https://brunch.co.kr/@hyeminimi/31)

* * *
# 참고 사이트

- https://docs.spring.io/autorepo/docs/spring-boot/current/reference/htmlsingle/#boot-features-error-handling-custom-error-pages

- https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc

- https://www.thymeleaf.org/doc/articles/springsecurity.html

- https://velog.io/@godori/spring-boot-error

- https://eblo.tistory.com/50