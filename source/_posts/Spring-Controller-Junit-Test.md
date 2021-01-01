---
layout: posts
title: 'Spring Controller Junit으로 Test하기'
date: 2020-05-30 15:14:57
tags:
    - Spring
    - Junit
    - Test
---

평소 Junit의 assertEquals로 알고리즘 코드만 test 하다가
토이 프로젝트에서도 Junit을 사용해보기 위해 Spring에서의 Junit Test 방법을 정리 해보자 한다

* * *

## Test할 Controller

```java
@Controller
public class HomeController {

    @GetMapping("/")
    public String home(Locale locale, Model model) {
        Date date = new Date();
        DateFormat dateFormat = DateFormat.getDateTimeInstance(DateFormat.LONG, DateFormat.LONG, locale);

        String formattedDate = dateFormat.format(date);

        model.addAttribute("serverTime", formattedDate );

        return "home";
    }
}
```

- index 페이지로 매핑되는 home 메서드만 존재하는 Controller

* * *

## standaloneSetup() 메서드를 활용한 Test

```java
public class HomeControllerTest {

    private MockMvc mockMvc;

    @Before
    public void setUp() {
        this.mockMvc = MockMvcBuilders.standaloneSetup(new HomeController()).build();
    }

    @Test
    public void homeTest() throws Exception {

        mockMvc.perform(get("/"))
        .andExpect(status().isOk())
        .andExpect(model().attributeExists("serverTime"))
        .andExpect(view().name("home"));
    }
}
```

MockMvc 클래스를 통해 HTTP GET, POST 등에 대한 테스트를 할 수 있게 한다

- perform() : get 방식으로 url을 호출
- status() : http status에 대해 테스트
- model() : model에 "serverTime" attribute가 존재하는 지 확인
- view() : return하는 view의 name이 "home"인지 확인

위 메서드들을 정상적으로 사용하기 위해선 아래 처럼 메서드들을 import 시켜야한다

```java
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.model;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.view;
```

* * *

## webAppContextSetup() 메서드를 활용한 Test

```java
@RunWith(SpringRunner.class)
@ContextConfiguration(locations = {"file:src/main/webapp/WEB-INF/spring/appServlet/servlet-context.xml"
        , "file:src/main/webapp/WEB-INF/spring/root-context.xml"
        , "file:src/main/webapp/WEB-INF/spring/spring-security.xml"})
@WebAppConfiguration
public class HomeControllerTest {

    @Autowired
    private WebApplicationContext context;

    private MockMvc mockMvc;

    @Before
    public void setUp() {
        this.mockMvc = MockMvcBuilders.webAppContextSetup(this.context).build();
    }

    @Test
    public void homeTest() throws Exception {

        mockMvc.perform(get("/"))
        .andExpect(status().isOk())
        .andExpect(model().attributeExists("serverTime"))
        .andExpect(view().name("home"));
    }
}
```

### @RunWith

- Junit에 내장된 Runner외에 다른 실행자를 실행시킨다
- 위 코드에선 SpringRunner 실행자로 Test를 실행하는데 이때 SpringRunner class는 SpringJUnit4ClassRunner class를 상속받고 있다
- 테스트를 진행할 때 ApplicationContext를 만들고 관리하는데 이 때 싱글톤을 보장한다

### @ContextConfiguration

- 스프링 Bean 설정 파일의 위치를 지정한다
- 위 코드에서는 Controller는 servlet-context.xml, 그 외 Service, Repository 등은 root-context.xml, 암호화를 위한 bcryptPasswordEncoder bean은 spring-security.xml에 지정되어 있어 3개를 지정했다

### @WebAppConfiguration

- applicationContext의 웹 버전을 작성하는데 사용된다

* * *

## standalonesetup과 webAppContextSetup의 차이

standalonesetup() 메서드는 테스트할 컨트롤러를 수동으로 초기화해서 주입하고, webAppContextSetup() 메서드는 스프링의 ApplicationContext의 인스턴스로 동작한다

standalonesetup() 메서드는 컨트롤러에 집중하여 테스트할 때 사용되고
webAppContextSetup() 메서드는 스프링의 DI 컨테이너를 이용해 스프링 MVC 동작을 재현해서 테스트한다

* * *

## 번외 java.lang.NoClassDefFoundError: javax/servlet/SessionCookieConfig 에러

```java
java.lang.NoClassDefFoundError: javax/servlet/SessionCookieConfig
    at org.springframework.test.web.servlet.setup.StandaloneMockMvcBuilder.initWebAppContext(StandaloneMockMvcBuilder.java:339)
    at org.springframework.test.web.servlet.setup.AbstractMockMvcBuilder.build(AbstractMockMvcBuilder.java:139)
    at com.toon.controller.HomeControllerTest.setUp(HomeControllerTest.java:25)
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    at sun.reflect.NativeMethodAccessorImpl.invoke(Unknown Source)
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(Unknown Source)
    at java.lang.reflect.Method.invoke(Unknown Source)
    at org.junit.runners.model.FrameworkMethod$1.runReflectiveCall(FrameworkMethod.java:50)
    at org.junit.internal.runners.model.ReflectiveCallable.run(ReflectiveCallable.java:12)
    at org.junit.runners.model.FrameworkMethod.invokeExplosively(FrameworkMethod.java:47)
    at org.junit.internal.runners.statements.RunBefores.evaluate(RunBefores.java:24)
    ... 26 more
```

- Spring 4.0 이상에서 Junit으로 테스트 시 servlet 3.0 API를 사용하기 때문에 발생
- pom.xml에서 servlet 버전을 3.0.1 이상으로 올려주면 해결
- artifactId가 __javax.servlet-api__ 으로 변경 됐으므로 함께 변경해준다.

```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.1.0</version>
    <scope>provided</scope>
</dependency>
```

## 참고 사이트

- [https://itmore.tistory.com/entry/MockMvc-%EC%83%81%EC%84%B8%EC%84%A4%EB%AA%85](https://itmore.tistory.com/entry/MockMvc-%EC%83%81%EC%84%B8%EC%84%A4%EB%AA%85)
- [https://jdm.kr/blog/165](https://jdm.kr/blog/165)
- [https://effectivesquid.tistory.com/entry/Spring-test-%EC%99%80-Junit4%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%ED%85%8C%EC%8A%A4%ED%8A%B8](https://effectivesquid.tistory.com/entry/Spring-test-%EC%99%80-Junit4%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%ED%85%8C%EC%8A%A4%ED%8A%B8)
- [https://www.baeldung.com/spring-webappconfiguration](https://www.baeldung.com/spring-webappconfiguration)