---
layout: posts
title: content-type application/x-www-form-urlencoded는 POST방식으로만
date: 2021-11-21 22:18:58
tags:
    - servlet
    - spring
    - 강의
---

## 서론

오늘도 어김없이 영한님의 스프링 MVC 강의를 듣고 있었는데, html form 데이터를 body에 보낼 때는 post 방식 밖에 안된다는 말씀을 하시기에... 
'그럼 GET 방식도 안된다는건가?' 라는 궁금증이 들어서 직접 확인해본 결과에 대해 작성한다.

* * *

## 본론

우선... 기본적으로 지금 필요한 GET방식과 POST방식의 차이점만을 다시 상기해보자면, GET방식은 데이터가 URL의 query string으로 들어간다는 것과,
POST방식은 데이터가 http message-body에 들어간다는 차이만 알아두면 될 것 같다. (다른 많은 차이가 있지만)

그렇기 때문에 우리가 html form 태그 에서 method attribute에 GET라고 작성한 후 데이터를 submit하면 데이터는 query string에 담겨 서버로 전송되고
POST라고 작성한 후 데이터를 submit하면 데이터는 message-body에 담겨서 서버로 전성되고 이 때 content-type은 application/x-www-form-urlencoded로 전송된다.

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form action="/request-param" method="post">
    username: <input type="text" name="username" />
    age:      <input type="text" name="age" />
    <button type="submit">전송</button>
</form>
</body>
</html>
```

{% asset_img 1.JPG content-type application/x-www-form-urlencoded는 POST방식으로만 %}

그럼 method는 GET이면서 content-type은 application/x-www-form-urlencoded 일 경우엔 어떻게 될까?

postman으로 아래처럼 실행해본 결과 보는 것과 같이 400 error가 응답되며 요청에 실패하였다.

{% asset_img 2.JPG content-type application/x-www-form-urlencoded는 POST방식으로만 %}

어째서 이런 일이 발생하였나를 코드 레벨에서 확인해보니...

tomcat의 Request class에 있는 [parseParameters()](https://tomcat.apache.org/tomcat-8.5-doc/api/org/apache/catalina/connector/Request.html#parseParameters())라는 메서드에서 넘어온 데이터를 parsing 하는데,
이 때 http method가 post인지 확인하고 post가 아니라면 message-body의 데이터를 파싱 할 수 없게 되어 있는 것을 확인했다.

```java
if( !getConnector().isParseBodyMethod(getMethod()) ) {
    success = true;
    return;
}

...


// 이 아래부턴 Connector.java

protected boolean isParseBodyMethod(String method) {
    return parseBodyMethodsSet.contains(method);
}

...

if (null == parseBodyMethodsSet) {
    setParseBodyMethods(getParseBodyMethods());
}

...

/**
 * Set list of HTTP methods which should allow body parameter
 * parsing. This defaults to <code>POST</code>.
 *
 * @param methods Comma separated list of HTTP method names
 */
public void setParseBodyMethods(String methods) {

    HashSet<String> methodSet = new HashSet<>();

    if (null != methods) {
        methodSet.addAll(Arrays.asList(methods.split("\\s*,\\s*")));
    }

    if (methodSet.contains("TRACE")) {
        throw new IllegalArgumentException(sm.getString("coyoteConnector.parseBodyMethodNoTrace"));
    }

    this.parseBodyMethods = methods;
    this.parseBodyMethodsSet = methodSet;
    setProperty("parseBodyMethods", methods);
}

...

/**
 * @return the HTTP methods which will support body parameters parsing
 */
public String getParseBodyMethods() {
    return this.parseBodyMethods;
}

...


/**
 * Comma-separated list of HTTP methods that will be parsed according
 * to POST-style rules for application/x-www-form-urlencoded request bodies.
 */
protected String parseBodyMethods = "POST";
```

위에 정리한 것 처럼

  1. isParseBodyMethod()에서 지금 request의 method가 body를 파싱 할 수 있는 메서드 인지 확인하는데, 이 때 가능한 method들은 인스턴스 변수 parseBodyMethodsSet에 저장되어 있고,
  2. 인스턴스 변수 parseBodyMethodsSet는 setParseBodyMethods() 메서드에서 저장되고 있고~
  3. parseBodyMethodsSet가 null인 경우에
  4. setParseBodyMethods(getParseBodyMethods()) 메서드를 실행해서 이 때 method를 저장 시키는데,
  5. getParseBodyMethods() 메서드를 호출하면
  6. 인스턴스변수 parseBodyMethods를 return 하는데
  7. 이 때 이 인스턴스 변수 parseBodyMethods에는 이미 "POST"로 method가 지정되어 있기 때문에
  8. GET방식 일 경우에는 message-body를 파싱 할 수 있는 method가 아닌 것으로 처리 되므로~
  9. 파싱 할 수 없다는 에러 메시지를 출력하면서 프로세스가 끝이난다.

* * *

## 결론

생각해보면... content-type은 message-body이 어떤 형식의 데이터로 담겨 있는가를 알려주는 일종의 os의 파일 확장자같은 개념인데
당연히 get방식의 경우 message-body에 데이터가 담기는 것이 아니니 get방식으로 content-type만 application/x-www-form-urlencoded로 한다고 될 턱이 없는데
강의를 듣기만 했을 때는 왜 생각하지 못했는 지 모르겠다.. 머리가 나쁘면 몸이 고생이라더니 옛말에 틀린 말이 없다.
* * *

## 참고 사이트

- [https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/Content-Type](https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/Content-Type)
- [https://datatracker.ietf.org/doc/html/rfc7231#section-3.1.1.5](https://datatracker.ietf.org/doc/html/rfc7231#section-3.1.1.5)
- [https://www.w3.org/Protocols/rfc2616/rfc2616-sec4.html](https://www.w3.org/Protocols/rfc2616/rfc2616-sec4.html)
