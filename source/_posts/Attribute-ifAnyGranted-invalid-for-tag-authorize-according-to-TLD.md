---
layout: posts
title: 'Attribute [ifAnyGranted] invalid for tag [authorize] according to TLD'
date: 2021-08-01 18:18:55
tags:
    - spring
---

## 상황

spring으로 된 프로젝트를 spring boot으로 전환 중에 spring-security-taglibs의 버전을 3.x 에서 5.5.1로 업데이트 했는데 아래와 같은 에러가 발생하여 이에 관한 내용을 정리 해둔다.

* * *

## 발생한 에러

```java
org.apache.jasper.JasperException: /WEB-INF/views/layout/layout.jsp (line: [126], column: [0]) Attribute [ifAnyGranted] invalid for tag [authorize] according to TLD
        at org.apache.jasper.compiler.DefaultErrorHandler.jspError(DefaultErrorHandler.java:41) ~[tomcat-embed-jasper-9.0.48.jar!/:na]
        at org.apache.jasper.compiler.ErrorDispatcher.dispatch(ErrorDispatcher.java:292) ~[tomcat-embed-jasper-9.0.48.jar!/:na]
        at org.apache.jasper.compiler.ErrorDispatcher.jspError(ErrorDispatcher.java:115) ~[tomcat-embed-jasper-9.0.48.jar!/:na]
        at org.apache.jasper.compiler.Validator$ValidateVisitor.checkXmlAttributes(Validator.java:1287) ~[tomcat-embed-jasper-9.0.48.jar!/:na]
        at org.apache.jasper.compiler.Validator$ValidateVisitor.visit(Validator.java:900) ~[tomcat-embed-jasper-9.0.48.jar!/:na]
        at org.apache.jasper.compiler.Node$CustomTag.accept(Node.java:1558) ~[tomcat-embed-jasper-9.0.48.jar!/:na]
        at org.apache.jasper.compiler.Node$Nodes.visit(Node.java:2385) ~[tomcat-embed-jasper-9.0.48.jar!/:na]
        at org.apache.jasper.compiler.Node$Visitor.visitBody(Node.java:2437) ~[tomcat-embed-jasper-9.0.48.jar!/:na]
        at org.apache.jasper.compiler.Node$Visitor.visit(Node.java:2443) ~[tomcat-embed-jasper-9.0.48.jar!/:na]
        at org.apache.jasper.compiler.Node$Root.accept(Node.java:471) ~[tomcat-embed-jasper-9.0.48.jar!/:na]
        at org.apache.jasper.compiler.Node$Nodes.visit(Node.java:2385) ~[tomcat-embed-jasper-9.0.48.jar!/:na]

        ...
```

## 왜 발생했을까?

{% blockquote spring-security, https://docs.spring.io/spring-security/site/migrate/current/3-to-4/html5/migrate-3-to-4-jc.html#m3to4-deprecations-taglibs Migrating from Spring Security 3.x to 4.x %}

This section describes all of the deprecated APIs within the spring-security-taglibs module. If you are not using the spring-security-taglibs module or have already completed this task, you can safely skip to spring-security-web.

Spring Security’s authorize JSP tag deprecated the properties ifAllGranted, ifAnyGranted, and ifNotGranted in favor of using expressions.
{% endblockquote %}


문서에 적혀 있는 것 처럼 spring-security-taglibs 4.x 버전 이상 부터는 ifAllGranted, ifAnyGranted, ifNotGranted 속성들을 지원하지 않기 때문에 3.x 버전에서 5 버전으로 업그레이드하면서 해당 속성들을 사용한 코드들이 있는 경우 발생하는 문제였다.

## 해결방법

```jsp

<sec:authorize ifAllGranted="ROLE_ADMIN,ROLE_USER">
    <p>Must have ROLE_ADMIN and ROLE_USER</p>
</sec:authorize>
<sec:authorize ifAnyGranted="ROLE_ADMIN,ROLE_USER">
    <p>Must have ROLE_ADMIN or ROLE_USER</p>
</sec:authorize>
<sec:authorize ifNotGranted="ROLE_ADMIN,ROLE_USER">
    <p>Must not have ROLE_ADMIN or ROLE_USER</p>
</sec:authorize>

```

위 처럼 되어 있을 경우

```jsp


<sec:authorize access="hasRole('ROLE_ADMIN') and hasRole('ROLE_USER')">
    <p>Must have ROLE_ADMIN and ROLE_USER</p>
</sec:authorize>
<sec:authorize access="hasAnyRole('ROLE_ADMIN','ROLE_USER')">
    <p>Must have ROLE_ADMIN or ROLE_USER</p>
</sec:authorize>
<sec:authorize access="!hasAnyRole('ROLE_ADMIN','ROLE_USER')">
    <p>Must not have ROLE_ADMIN or ROLE_USER</p>
</sec:authorize>

```

아래 처럼 치환해주면 정상적으로 작동하는 것을 확인 할 수 있을 것이다.

* * *

## 참고 사이트

- [https://okky.kr/article/679832](https://okky.kr/article/679832)
- [https://docs.spring.io/spring-security/site/migrate/current/3-to-4/html5/migrate-3-to-4-jc.html#m3to4-deprecations-taglibs](https://docs.spring.io/spring-security/site/migrate/current/3-to-4/html5/migrate-3-to-4-jc.html#m3to4-deprecations-taglibs)
