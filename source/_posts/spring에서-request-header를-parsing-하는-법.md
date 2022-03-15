---
layout: posts
title: spring에서 request header를 parsing 하는 법
date: 2021-11-15 21:44:15
tags:
    - spring
    - tomcat
---

## 서론

우선 사과의 말씀부터 올린다. 제목은 'spring에서~' 지만 정확히는 tomcat에서 ~ 로 보는 것이 맞다.

인프런에서 [김영한 님의 Spring MVC 강의](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-1/dashboard)를 보던 도중 ... 'http request를 정확히 어떠한 방식으로 파싱하고 있을까?'라는 궁금증이 생겨서 강의를 듣다가 말고 삽질을 한 내용을 기록해둔다.
* * *

## 본론

우선, 처음으로 궁금증을 해결하기 위해 들어가본 곳은 http request 요청을 받기 위한 [HttpServletRequest interface](https://javaee.github.io/javaee-spec/javadocs/javax/servlet/http/HttpServletRequest.html)의 tomcat 구현체인 [RequestFacade](https://tomcat.apache.org/tomcat-8.0-doc/api/org/apache/catalina/connector/RequestFacade.html) class부터 시작했다.

이 중 getHeader 메서드에 대해 찾아보면,

{% blockquote RequestFacade https://tomcat.apache.org/tomcat-8.0-doc/api/org/apache/catalina/connector/RequestFacade.html#getHeader(java.lang.String) RequestFacade %}
Returns the value of the specified request header as a String.
If the request did not include a header of the specified name, this method returns null.
If there are multiple headers with the same name, this method returns the first head in the request.
The header name is case insensitive.
You can use this method with any request header.
{% endblockquote %}

라고 설명되어 있고... 이 때 코드에서 request의 header를 살펴보면

{% asset_img 1.JPG spring에서 request header를 parsing 하는 법 "이미 header에 정보들이 저장되어 있는 상태" "'이미 header에 정보들이 저장되어 있는 상태'" %}

이미지와 같이 이미 header에 대한 정보들이 각각 저정되어 있는 걸 볼 수 있다.

필자가 궁금했던 것은 '이 request에 header는 언제 저장되는가?' 였으므로
막연하게 '여기서 더 타고 올라가면 찾아 낼 수 있겠구나..' 라고 안일한(?) 생각을 하고 안으로 더 들어가보았다.

타고타고 가다보니 ...
http 1.1 header에 의한 요청이므로 Http11Processor의 service 메서드에서

```java
// Don't parse headers for HTTP/0.9
if (!http09 && !inputBuffer.parseHeaders()) {
    // We've read part of the request, don't recycle it
    // instead associate it with the socket
    openSocket = true;
    readComplete = false;
    break;
}
```

처럼 작성되어 있는 코드를 발견했고, 여기에 있는 [inputBuffer](https://tomcat.apache.org/tomcat-9.0-doc/api/org/apache/coyote/http11/Http11InputBuffer.html).parseHeaders() 을 보니
이름부터 여기서 http request에 대해 파싱을 할 것 같아 이 안으로 들어가보았다.

```java
while (headerParsePos == HeaderParsePosition.HEADER_NAME) {

    // Read new bytes if needed
    if (byteBuffer.position() >= byteBuffer.limit()) {
        if (!fill(false)) { // parse header
            return HeaderParseStatus.NEED_MORE_DATA;
        }
    }

    int pos = byteBuffer.position();
    chr = byteBuffer.get();
    if (chr == Constants.COLON) {
        headerParsePos = HeaderParsePosition.HEADER_VALUE_START;
        headerData.headerValue = headers.addValue(byteBuffer.array(), headerData.start,
                pos - headerData.start);
        pos = byteBuffer.position();
        // Mark the current buffer position
        headerData.start = pos;
        headerData.realPos = pos;
        headerData.lastSignificantChar = pos;
        break;
    } else if (!HttpParser.isToken(chr)) {
        // Non-token characters are illegal in header names
        // Parsing continues so the error can be reported in context
        headerData.lastSignificantChar = pos;
        byteBuffer.position(byteBuffer.position() - 1);
        // skipLine() will handle the error
        return skipLine();
    }

    // chr is next byte of header name. Convert to lowercase.
    if ((chr >= Constants.A) && (chr <= Constants.Z)) {
        byteBuffer.put(pos, (byte) (chr - Constants.LC_OFFSET));
    }
}
```

들어가서보니 해당 코드처럼

현재 header data의 파싱된 위치가 key, value 형식의 데이터인 header의 name 부분 이라면 계속 반복하면서 버퍼에 저장하고 COLON(:) 을 만나게 되면,

```java
headerData.headerValue = headers.addValue(byteBuffer.array(), headerData.start,
                        pos - headerData.start);
```

코드가 실행되면서 아까 봤던 request의 MimeHeaders 타입인 header에 name으로 저장되고

value에도 위와 같은 방식으로 byte 단위로 이동하고 저장되면서 header에 value에 저장된다.
* * *

## 결론

서블릿이 http 통신을 위해 귀찮은 것들을 해준다는 것은 알고 있었지만
실제로 어떻게 하고 지에 대해서는 신경을 안 쓰고 있었는데 이렇게 강의를 듣다 보니 궁금해서 한번 확인해보았다.

만일 이 작업을 서블릿에서 대신 안해주지 않고 개발자가 개발할 때마다 해줘야한다고 생각하면... 
머리가 어지러워지는 것 같아 이만 글을 줄인다...
* * *

## 참고 사이트

- [스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-1/dashboard)