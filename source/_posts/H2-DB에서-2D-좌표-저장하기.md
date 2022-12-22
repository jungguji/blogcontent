---
layout: posts
title: H2 DB에서 2D 좌표 저장하기
date: 2022-11-30 20:27:13
tags:
    - H2
---

## 서론

 [토이 프로젝트](https://github.com/jungguji/sokdak-sokdak)를 진행 중, 장소의 위치의 위도, 경도 좌표를 저장해야 했는데, float type의 'latitude', 'longitude'라는 2개의 컬럼을 만들어서 각각 따로 저장하려고 했는데, 우연찮은 기회에 [공간 데이터 타입](https://dev.mysql.com/doc/refman/8.0/en/spatial-type-overview.html)이라는 것이 따로 있다는 것을 발견하여 적용해본 경험을 글로 작성하여 보려고 한다.

## 본론

 {% blockquote 위키백과 https://ko.wikipedia.org/wiki/%EA%B3%B5%EA%B0%84 공간 %}
    공간(空間, 영어: space)은 어떤 물질 또는 물체가 존재할 수 있거나 어떤 일이 일어날 수 있는 장소이다.
 {% endblockquote %}

<br />

공간 데이터란, 위 같은 점, 선, 면의 공간을 데이터화 한 것을 의미하는데, 필자는 지도에서 위치를 표시하기 위해 2D 공간 데이터인 점(Point) 데이터를 저장할 필요가 있었다.
필자는 Spring boot + JPA(Hibernate) + H2 환경을 사용하고 있어서, 공간 데이터를 다루기 위해 [hibernate-spatial](https://docs.jboss.org/hibernate/orm/5.2/userguide/html_single/Hibernate_User_Guide.html#spatial) 의존성을 추가해주었다.

<br />

```xml
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-spatial</artifactId>
    <version>${hibernate.version}</version>
</dependency>
```

<br />

그 후, hibernate-spatial 라이브러리에 포함된 [The JTS Topology Suite](https://github.com/locationtech/jts)를 사용해 좌표 데이터를 다루면 되는데, 위에서 말한 것처럼 필자는 2D 데이터를 다루기 위해 [Point](https://locationtech.github.io/jts/javadoc/org/locationtech/jts/geom/Point.html)를 사용했다.

<br />

```java
import javax.persistence.Column;
import javax.persistence.Embeddable;
import lombok.AccessLevel;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;
import org.locationtech.jts.geom.Point;

@Embeddable
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Address {

    @Column(name = "road")
    private String road;

    @Column(name = "jibun")
    private String jibun;

    @Column(name = "zip", nullable = false)
    private String zip;

    @Column(name = "location", nullable = false)
    private Point location;

    @Builder
    public Address(String road, String jibun, String zip, Point location) {
        this.road = road;
        this.jibun = jibun;
        this.zip = zip;
        this.location = location;
    }
}
```

- 만약 필자처럼 Point를 사용할 경우 org.springframework.data.geo.Point를 import하면 정상적으로 동작하지 않으므로, 꼭 import문을 확인하길 바란다.

위처럼 당차게 Entity class를 작성하고, 아래 같은 테스트 데이터를 넣고, 호기롭게 테스트를 돌려본 결과...

### 입력 데이터

```java

2022-11-30 22:48:44.230 TRACE 21788 --- [           main] o.h.type.descriptor.sql.BasicBinder      : binding parameter [1] as [TIMESTAMP] - [null]
2022-11-30 22:48:44.231 TRACE 21788 --- [           main] o.h.type.descriptor.sql.BasicBinder      : binding parameter [2] as [TIMESTAMP] - [null]
2022-11-30 22:48:44.231 TRACE 21788 --- [           main] o.h.type.descriptor.sql.BasicBinder      : binding parameter [3] as [VARCHAR] - [지번 서울시]
2022-11-30 22:48:44.232 TRACE 21788 --- [           main] o.h.type.descriptor.sql.BasicBinder      : binding parameter [4] as [VARBINARY] - [POINT (19 2)]
2022-11-30 22:48:44.233 TRACE 21788 --- [           main] o.h.type.descriptor.sql.BasicBinder      : binding parameter [5] as [VARCHAR] - [도로명 서울시]
2022-11-30 22:48:44.234 TRACE 21788 --- [           main] o.h.type.descriptor.sql.BasicBinder      : binding parameter [6] as [VARCHAR] - [01006]
2022-11-30 22:48:44.235 TRACE 21788 --- [           main] o.h.type.descriptor.sql.BasicBinder      : binding parameter [7] as [BIGINT] - [null]
2022-11-30 22:48:44.235 TRACE 21788 --- [           main] o.h.type.descriptor.sql.BasicBinder      : binding parameter [8] as [VARCHAR] - [우리집]

```

- 위 로그처럼 sql에 입력된 파라미터들이 보고 싶은 경우 아래처럼 설정을 추가해주면 된다.

```yaml
logging:
  level:
    org:
      hibernate:
        type:
          descriptor:
            sql: trace
```


### 발생한 에러 로그

```java
Caused by: org.h2.jdbc.JdbcSQLDataException: Value too long for column "LOCATION BINARY VARYING(255)": "X'aced00057372001f6f72672e6c6f636174696f6e746563682e6a74732e67656f6d2e506f696e74... (1151)"; SQL statement:
insert into place (id, create_date, update_date, jibun, location, road, zip, category_id, name) values (default, ?, ?, ?, ?, ?, ?, ?, ?) [22001-214]
	at org.h2.message.DbException.getJdbcSQLException(DbException.java:506)
	at org.h2.message.DbException.getJdbcSQLException(DbException.java:477)
	at org.h2.message.DbException.get(DbException.java:223)
	at org.h2.message.DbException.getValueTooLongException(DbException.java:322)
	at org.h2.value.Value.getValueTooLongException(Value.java:2573)
	at org.h2.value.Value.convertToVarbinary(Value.java:1371)
	at org.h2.value.Value.convertTo(Value.java:1125)
	at org.h2.value.Value.convertForAssignTo(Value.java:1092)
	at org.h2.table.Column.validateConvertUpdateSequence(Column.java:369)
	at org.h2.table.Table.convertInsertRow(Table.java:926)
	at org.h2.command.dml.Insert.insertRows(Insert.java:167)
	at org.h2.command.dml.Insert.update(Insert.java:135)
	at org.h2.command.CommandContainer.executeUpdateWithGeneratedKeys(CommandContainer.java:242)
	at org.h2.command.CommandContainer.update(CommandContainer.java:163)
	at org.h2.command.Command.executeUpdate(Command.java:252)
	at org.h2.jdbc.JdbcPreparedStatement.executeUpdateInternal(JdbcPreparedStatement.java:209)
	at org.h2.jdbc.JdbcPreparedStatement.executeUpdate(JdbcPreparedStatement.java:169)
	at com.zaxxer.hikari.pool.ProxyPreparedStatement.executeUpdate(ProxyPreparedStatement.java:61)
	at com.zaxxer.hikari.pool.HikariProxyPreparedStatement.executeUpdate(HikariProxyPreparedStatement.java)
	at org.hibernate.engine.jdbc.internal.ResultSetReturnImpl.executeUpdate(ResultSetReturnImpl.java:197)
	... 136 more
```

<br />

### 테이블 생성 SQL

```sql
create table place (
    id bigint generated by default as identity,
    create_date timestamp,
    update_date timestamp,
    jibun varchar(255),
    location varbinary(255) not null,
    road varchar(255),
    zip varchar(255) not null,
    category_id bigint,
    name varchar(255) not null,
    primary key (id)
)
```

이처럼 예상하지 못한 결과가 발생하였는데... 로그를 보니, table create 시 좌표를 저장하는 컬럼의 타입이 기대했던 [GEOMETRY](http://www.h2database.com/html/datatypes.html#geometry_type)이 아닌, varbinary 타입으로 생성되어서 발생한 에러인가 싶어서 컬럼 타입을 "GEOMETRY"로 지정하고 생성해봤지만, 결과는 마찬가지였다.

### 수정한 코드 및 실행된 SQL

```java

...

// columnDefinition 속성으로 컬럼 타입 지정
@Column(name = "location", nullable = false, columnDefinition = "GEOMETRY") 
private Point location;

...

```

<br />

```sql
create table place (
    id bigint generated by default as identity,
    create_date timestamp,
    update_date timestamp,
    jibun varchar(255),
    location GEOMETRY not null, -- 정상적으로 컬럼의 데이터 타입이 GEOMETRY로 변경
    road varchar(255),
    zip varchar(255) not null,
    category_id bigint,
    name varchar(255) not null,
    primary key (id)
)
```

다시 원인을 검색해본 결과... hibernate-spatial에서는 H2가 아닌 H2의 확장 데이터베이스인 [GeoDB](https://github.com/jdeolive/geodb)를 지원한다고 되어있는 것을 확인할 수 있었다.


{% blockquote Hibernate Docs https://docs.jboss.org/hibernate/orm/5.2/userguide/html_single/Hibernate_User_Guide.html#spatial GeoDB (H2) %}
    The GeoDBDialect supports the GeoDB a spatial extension of the H2 in-memory database.
{% endblockquote %}

그 후 GeoDB를 적용하기 위해 properties 혹은 yaml에 아래처럼 dialect 설정에 org.hibernate.spatial.dialect.h2geodb.GeoDBDialect를 추가해 주면,

```yaml
spring:
  datasource:
    url: jdbc:h2:mem:test
    username: sa
    password:
    driver-class-name: org.h2.Driver

  jpa:
    database-platform: org.hibernate.dialect.H2Dialect
    defer-datasource-initialization: true
    hibernate:
      ddl-auto: create-drop
    show-sql: true
    properties:
      hibernate:
        dialect: org.hibernate.spatial.dialect.h2geodb.GeoDBDialect # 추가 혹은 변경
```

### dialect 설정 전 dialect

```java
2022-11-30 23:37:47.975  INFO 15720 --- [           main] org.hibernate.dialect.Dialect            : HHH000400: Using dialect: org.hibernate.dialect.H2Dialect
```

### dialect 설정 후 dialect


```java
2022-11-30 23:35:42.124  INFO 18400 --- [           main] org.hibernate.dialect.Dialect            : HHH000400: Using dialect: org.hibernate.spatial.dialect.h2geodb.GeoDBDialect
```

<br />

{% asset_img "1_테스트성공.PNG" %}

<br />

이처럼 테스트가 성공하는 모습을  확인할 수 있었다.
* * *

## 참고 사이트

- [https://stackoverflow.com/questions/44261899/geometry-datatype-in-h2-w-spring-jpa](https://stackoverflow.com/questions/44261899/geometry-datatype-in-h2-w-spring-jpa)


