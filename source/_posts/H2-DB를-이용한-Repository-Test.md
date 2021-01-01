---
layout: posts
title: H2 DB를 이용한 Repository Test
date: 2020-07-05 15:56:17
tags:
    - Junit
    - Test
    - H2
---

## 상황

프로덕션 환경에서 Mariadb를 활용하고 있었는데, 프로덕션 환경의 DB로 Repository class를 테스트를 진행하니 테스트 데이터는 롤백되어 DB 남지 않았지만, Auto increment로 지정한 ID가 증가돼서 실제 서비스에서 데이터 저장 시 증가된 ID로 데이터가 저장되는 문제가 발생했다.

해서 Test할 때는 사용 할 데이터베이스 [H2](https://www.h2database.com/html/main.html)를 이용하기로 했다. \
H2 데이터베이스는 인메모리 관계형 데이터베이스로 메모리 안에서 실행되기 때문에 어플리케이션을 시작할 때마다 초기화되어 테스트 용도로 많이 사용된다. \
하지만 테스트 환경도 프로덕션 환경과 비슷하게 만들어서 테스트 하는 경우에는 테스트환경에도 프로덕션 DB를 생성해서 사용하는 경우도 있다고 한다.

필자가 okky에 올린 질문글 \
[Repository Test시 ID 자동 증가](https://okky.kr/article/732917?note=2011578)

* * *

## Test 환경에서 H2 적용

우선 H2 DB를 POM.xml에 추가하여 의존성을 등록한다.

```xml
<!-- https://mvnrepository.com/artifact/com.h2database/h2 -->
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>test</scope>
</dependency>
```

그 후 Test 환경에서 사용하는 appication.properties에서 데이터베이스 설정을 H2로 설정해준다.

<br />

![application-test](/images/20200705/application-test.PNG)

<br />

### application-test.properties 내용

```properties
spring.datasource.url=jdbc:h2:mem:testdb;MODE=MySQL;DB_CLOSE_DELAY=-1
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
spring.h2.console.enabled=true

spring.jpa.hibernate.dialect=org.hibernate.dialect.MariaDBDialect
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=create-drop
```

- 프로덕션 환경에서 Mariadb를 사용하기 때문에 dialect를 Mariadb로 설정하고 MODE=Mysql로 설정했다.
- spring.jpa.hibernate.ddl-auto=none으로 설정하면 시작 시 마다 초기화되지 않기 떄문에 테스트 환경에선 꼭 create-drop으로 설정해준다.

### Repository Test class

```java
@ExtendWith(SpringExtension.class)
@DataJpaTest
@TestPropertySource("classpath:application-test.properties")
class WordRepositoryTest {
    ...
}
```

- JPA Test를 위해 JPA 관련된 설정을 불러오는 @DataJpaTest
- Test환경에선 프로덕션 환경과 다르게 H2 DB를 사용하므로 H2 DB설정을 지정한 application-test.properties를 호출하기 위한 @TestPropertySource

이 정도로 설정하고 Test하면 정상적으로 H2를 이용한 테스트가 성공할 것이다.

* * *

## 여담

```java
@BeforeEach
void setUp() {
    user = new User();
    user.setUsername("jgji");
    user.setPassword("qwe123");

    userRepository.save(this.user);

    user1 = new User();
    user.setUsername("haha");
    user.setPassword("qwe123");

    userRepository.save(this.user1);

    List<Word> givenWordList = getWordGiven();
    this.word = givenWordList.get(0);
    this.word1 = givenWordList.get(1);
}
```

위 처럼 @Before 메서드를 지정해놓았는데, 각각 Test 메서드를 실행하였을 땐 Auto increment로 지정한 user의 id가 정상적으로 1, 2 이런식으로 생성되었지만 test class 전체로 test를 실행하니 DB가 메서드 마다 각각 실행하고 초기화 되는 것이 아닌지 User Id가 계속 증가해서 테스트가 실패하는 문제가 있었다.

테스트 시 꼭 메서드 각각으로하고 성공한다고 넘어가는게 아니라 클래스 전체로 테스트를 해봐야 할 것 같다.

* * *

## 프로젝트 전체 코드

- [https://docs.spring.io/spring-boot/docs/2.1.x/reference/html/howto-database-initialization.html](https://docs.spring.io/spring-boot/docs/2.1.x/reference/html/howto-database-initialization.html)

* * *

## 참고 사이트

- [https://docs.spring.io/spring-boot/docs/2.1.x/reference/html/howto-database-initialization.html](https://docs.spring.io/spring-boot/docs/2.1.x/reference/html/howto-database-initialization.html)
