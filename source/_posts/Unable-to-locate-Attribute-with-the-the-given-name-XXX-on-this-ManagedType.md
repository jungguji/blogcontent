---
layout: posts
title: 'Unable to locate Attribute  with the the given name [XXX] on this ManagedType'
date: 2021-09-12 16:02:14
tags:
    - Spring-Boot
---

## 서론

spring으로 된 프로젝트를 spring boot으로 전환 중... query method로 작성되어 있는 부분에서 발생한 문제에 대해 알아보고 이에 대한 해결 방법을 정리 해둔다.

* * *

## 본론

### 에러가 발생한 code

```java

public class XXXX {
    ... 

    private DDay dDay;

    ...
}

public interface Repository {
    ...

    List<Category> findByDDayOrderByDDayStartDateAsc(DDay dday);

    ...
}
    
```

### 발생한 에러 전문

```java
Caused by: java.lang.IllegalArgumentException: Unable to locate Attribute  with the the given name [attribute명] on this ManagedType [class명]
    at org.hibernate.metamodel.model.domain.internal.AbstractManagedType.checkNotNull(AbstractManagedType.java:148) ~[hibernate-core-5.4.32.Final.jar:5.4.32.Final]
    at org.hibernate.metamodel.model.domain.internal.AbstractManagedType.getAttribute(AbstractManagedType.java:119) ~[hibernate-core-5.4.32.Final.jar:5.4.32.Final]
    at org.hibernate.metamodel.model.domain.internal.AbstractManagedType.getAttribute(AbstractManagedType.java:44) ~[hibernate-core-5.4.32.Final.jar:5.4.32.Final]
    at org.springframework.data.jpa.repository.query.QueryUtils.requiresOuterJoin(QueryUtils.java:697) ~[spring-data-jpa-2.5.2.jar:2.5.2]
    at org.springframework.data.jpa.repository.query.QueryUtils.toExpressionRecursively(QueryUtils.java:638) ~[spring-data-jpa-2.5.2.jar:2.5.2]
    at org.springframework.data.jpa.repository.query.QueryUtils.toExpressionRecursively(QueryUtils.java:617) ~[spring-data-jpa-2.5.2.jar:2.5.2]
    at org.springframework.data.jpa.repository.query.QueryUtils.toExpressionRecursively(QueryUtils.java:613) ~[spring-data-jpa-2.5.2.jar:2.5.2]
    at org.springframework.data.jpa.repository.query.JpaQueryCreator$PredicateBuilder.getTypedPath(JpaQueryCreator.java:385) ~[spring-data-jpa-2.5.2.jar:2.5.2]
    at org.springframework.data.jpa.repository.query.JpaQueryCreator$PredicateBuilder.build(JpaQueryCreator.java:308) ~[spring-data-jpa-2.5.2.jar:2.5.2]
    at org.springframework.data.jpa.repository.query.JpaQueryCreator.toPredicate(JpaQueryCreator.java:211) ~[spring-data-jpa-2.5.2.jar:2.5.2]
    at org.springframework.data.jpa.repository.query.JpaQueryCreator.create(JpaQueryCreator.java:124) ~[spring-data-jpa-2.5.2.jar:2.5.2]
    at org.springframework.data.jpa.repository.query.JpaQueryCreator.create(JpaQueryCreator.java:59) ~[spring-data-jpa-2.5.2.jar:2.5.2]
    at org.springframework.data.repository.query.parser.AbstractQueryCreator.createCriteria(AbstractQueryCreator.java:119) ~[spring-data-commons-2.5.2.jar:2.5.2]
    at org.springframework.data.repository.query.parser.AbstractQueryCreator.createQuery(AbstractQueryCreator.java:95) ~[spring-data-commons-2.5.2.jar:2.5.2]
    at org.springframework.data.repository.query.parser.AbstractQueryCreator.createQuery(AbstractQueryCreator.java:81) ~[spring-data-commons-2.5.2.jar:2.5.2]
    at org.springframework.data.jpa.repository.query.PartTreeJpaQuery$QueryPreparer.<init>(PartTreeJpaQuery.java:217) ~[spring-data-jpa-2.5.2.jar:2.5.2]
    at org.springframework.data.jpa.repository.query.PartTreeJpaQuery$CountQueryPreparer.<init>(PartTreeJpaQuery.java:348) ~[spring-data-jpa-2.5.2.jar:2.5.2]
    at org.springframework.data.jpa.repository.query.PartTreeJpaQuery.<init>(PartTreeJpaQuery.java:91) ~[spring-data-jpa-2.5.2.jar:2.5.2]
    ... 117 common frames omitted


Process finished with exit code 0

```

### 왜 발생 했을까?

{% blockquote spring-data-commons https://github.com/spring-projects/spring-data-commons/issues/1996 spring-projects %}
The inconsistency between the subject part and the OrderBy part is certainly a bug.

For the given case the correct behaviour  seem to be the one from the OrderBy part which is adhering to the conventions laid out in the Java Bean Specification 1.0.1 Section 8.8:

...

There is no way to write the property name in the middle of a java method name if the property begins with a single lower case letter followed by an upper case letter.

Therefore renaming the property, or less invasive providing a @Query annotation are the ways to solve this problem on your side GitFlip
{% endblockquote %}

<br />

독자들의 빠른 문제 해결을 위해 결론부터 이야기 하자면 위의 글에서 보는 것 처럼 query method에서 OrderBy를 사용 할 때 "dDay" 같이 맨 앞 한 자리가 소문자 변수명을 사용 하는 경우  spring-data-jpa의 버그로 발생하는 에러인데 해결방법으로는 @Query annotation을 사용하거나 변수명을 변경하는 것을 권하고 있다.
* * *

## 결론

### 해결 방법

```java
@Query("SELECT c FROM Entity명 c WHERE c.dDay =:dday ORDER BY dDayStartDate")
List<Category> findByDDayOrderByDDayStartDateAsc(DDay dday);
```

위에서 알려준 해결책 중 필자는 @Query Annotation을 사용하는 방법으로 문제를 수정하였는데 다른 이유가 있는 것은 아니고 현재 프로젝트에서 변수명을 변경하는 것이 더 코스트가 많이 발생하기 때문에 @Query를 사용해서 처리했다.

이 문제 때문에 꽤 많은 시간을 소비 하였는데 필자와 같은 무고한 피해자(?)가 발생하지 않기를 바라며 글을 마친다.
* * *

## 참고 사이트

- [https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.query-methods.query-property-expressions](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.query-methods.query-property-expressions)
- [https://hongsii.github.io/2019/01/06/jpa-query-creation-with-underscore/](https://hongsii.github.io/2019/01/06/jpa-query-creation-with-underscore/)
- [https://github.com/spring-projects/spring-data-commons/issues/1996](https://github.com/spring-projects/spring-data-commons/issues/1996)
