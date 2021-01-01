---
layout: posts
title: '@SpringBootApplication의 동작 방식'
date: 2020-05-26 11:13:49
tags:
    - Spring-Boot
---

## @SpringBootApplication의 동작 방식

Spring-Boot의 Main Application 코드에는 @SpringBootApplication이라는 Annotation이 존재한다.

```java
@SpringBootApplication
public class Demo1Application {
   public static void main(String[] args) {
      SpringApplication.run(Demo1Application.class, args);
   }
}
```

- @SpringBootApplication

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
      @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {

    ...

}
```

이 Annotation의 주된 내용은 아래 3가지 이다.

- @SpringBootConfiguration
- @EnableAutoConfiguration
- @ComponentScan

* * *

## @SpringBootConfiguration

- [@SpringBootConfiguration](https://www.baeldung.com/springbootconfiguration-annotation)은 기존 @Configuration과 마찬가지로 해당 클래스가 @Bean 메서드를 정의되어 있음을 Spring 컨테이너에 알려주는 역할을 한다.
- [@Configuration](https://www.baeldung.com/spring-bean-annotations) 어노테이션의 대안으로 둘의 차이점은 아래와 같이 설명하고 있다.

{% blockquote baeldung, https://www.baeldung.com/springbootconfiguration-annotation#2-springbootconfiguration-vs-configuration Guide to @SpringBootConfiguration in Spring Boot %}
@SpringBootConfiguration is an alternative to the @Configuration annotation. The main difference is that @SpringBootConfiguration allows configuration to be automatically located. This can be especially useful for unit or integration tests.
{% endblockquote %}

## @EnableAutoConfiguration

- @ComponentScan에서 먼저 스캔해서 Bean으로 등록하고 tomcat등 스프링이 정의한 외부 의존성을 갖는 class들을 스캔해서 Bean으로 등록한다.
- 이 때 정의된 class들은 spring-boot-autoconfigure안에 있는 MATE-INF 폴더에 spring.factories라는 파일안에 정의되어 있다.
- 이 중에서 spring.boot.enableautoconfiguration을 key로하는 외부 의존성 class들이 존재한다.
- 여기에 정의된 모든 class를 가져오는 게 아니라 class에 내부에 정의된 어노테이션에 따라 그 조건에 부합하는 class들만 생성된다.
  - @ConditionalOnProperty, @ConditionalOnClass, @ConditionalOnBean 등등

```java
public @interface EnableAutoConfiguration {

   String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    ...
}
```

### spring.factories 위치

![spring-boot-autoconfigure](/images/20200526/spring-boot-autoconfigure.png)

![META-INF](/images/20200526/META-INF.png)

```text
...

# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
org.springframework.boot.autoconfigure.cloud.CloudServiceConnectorsAutoConfiguration,\
org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,\

...
```

### 번외. 내장 Tomcat Class

spring.factories 파일 안에 보면

```text
org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration,\
```

가 있고 이 class의 코드를 보면 아래와 같이 tomcatServletWebServerFactoryCustomizer() 메서드를 통해 tomcat을 실행한다는 것을 알 수 있다.

```java
@Import({ ServletWebServerFactoryAutoConfiguration.BeanPostProcessorsRegistrar.class,
        ServletWebServerFactoryConfiguration.EmbeddedTomcat.class,
        ServletWebServerFactoryConfiguration.EmbeddedJetty.class,
        ServletWebServerFactoryConfiguration.EmbeddedUndertow.class })
public class ServletWebServerFactoryAutoConfiguration {

    @Bean
    public ServletWebServerFactoryCustomizer servletWebServerFactoryCustomizer(ServerProperties serverProperties) {
        return new ServletWebServerFactoryCustomizer(serverProperties);
    }

    @Bean
    @ConditionalOnClass(name = "org.apache.catalina.startup.Tomcat")
    public TomcatServletWebServerFactoryCustomizer tomcatServletWebServerFactoryCustomizer(
            ServerProperties serverProperties) {
        return new TomcatServletWebServerFactoryCustomizer(serverProperties);
    }

...
```

## @ComponentScan

- @ComponentScan은 해당 어노테이션 하위에 있는 객체들 중 @Component가 선언된 클래스들을 찾아 Bean으로 등록하는 역할을 한다.
- 이 때 꼭 @Component가 아니여도 @Component가 선언되어 있는 어노테이션인 @Service, @Repository, @Controller 등등도 포함된다.
- @EnableAutoConfiguration이 스캔하기 전에 먼저 @ComponentScan이 진행된다.

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Service {
    ...
}

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Repository {
    ...
}

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Controller {
    ...
}
```

* * *

## 참고 사이트

- [https://www.youtube.com/watch?v=OXILjfY8edw&list=PLgXGHBqgT2TvpJ_p9L_yZKPifgdBOzdVH&index=7](https://www.youtube.com/watch?v=OXILjfY8edw&list=PLgXGHBqgT2TvpJ_p9L_yZKPifgdBOzdVH&index=7)
- [https://seongmun-hong.github.io/springboot/Spring-boot-EnableAutoConfiguration](https://seongmun-hong.github.io/springboot/Spring-boot-EnableAutoConfiguration)
- [http://dveamer.github.io/backend/SpringBootAutoConfiguration.html](http://dveamer.github.io/backend/SpringBootAutoConfiguration.html)
- [https://www.baeldung.com/springbootconfiguration-annotation](https://www.baeldung.com/springbootconfiguration-annotation)
- [http://wonwoo.ml/index.php/post/20](http://wonwoo.ml/index.php/post/20)