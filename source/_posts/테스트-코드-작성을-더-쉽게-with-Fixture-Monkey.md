---
layout: posts
title: 테스트 코드 작성을 더 쉽게 (with. Fixture Monkey)
date: 2024-11-19 23:16:40
tags:
    - test
---

# 1. **Fixture Monkey란?**

Fixture Monkey는 네이버의 내부 프로젝트인 **Plasma**에서 복잡한 테스트 요구 사항을 해결하기 위해 개발된 도구로, 현재는 오픈 소스로 제공되어 Java와 Kotlin 환경에서 쉽게 적용할 수 있습니다. 이 라이브러리는 기존의 고정된 테스트 데이터 대신  복잡한 객체 타입의 테스트 데이터를 자동으로 생성하여 테스트 코드의 일관성과 효율성을 대폭 향상시킬 수 있는 도구입니다.

--- 

# 2. Fixture Monkey 도입의 필요성

## 1. 클래스에 final 필드가 포함된 경우

```java
@Getter
@RequiredArgsConstructor
public class Member {
    private final long id;
    private final String name;
    private final String email;
    private final String phoneNumber;
    private final int age;
    private final boolean isVerified;
    private final LocalDateTime createdAt;

    public String getAgeCategory() {
        assert this.age >= 0;

        if (this.age < 18) {
            return "청소년";
        } else if (this.age < 65) {
            return "청년";
        } else {
            return "시니어";
        }
    }
}
```

### 문제점
- Member 클래스의 필드들이 모두 final로 선언되어 있기 때문에, 단순히 테스트에서 특정 메서드(getAgeCategory)만 검증하려 해도 불필요한 모든 데이터를 포함하여 객체를 생성해야 하는 문제가 있습니다.

```java
@Test
void testGetAgeCategory() {
    // 청소년
    Member minor = new Member(1L, "Alice", "alice@example.com", "123456789", 15, true, LocalDateTime.now());
    assertEquals("청소년", minor.getAgeCategory());

    // 청년
    Member adult = new Member(2L, "Bob", "bob@example.com", "987654321", 30, true, LocalDateTime.now());
    assertEquals("청년", adult.getAgeCategory());

    // 시니어
    Member senior = new Member(3L, "Charlie", "charlie@example.com", "555555555", 70, true, LocalDateTime.now());
    assertEquals("시니어", senior.getAgeCategory());
}
```

## 2. 하나의 클래스에 너무 많은 객체를 포함하는 경우

```java
@Getter
public class Order {
    private final long orderId;
    private final Customer customer; // 연관된 객체
    private final Product product;   // 연관된 객체
    private final LocalDateTime orderDate;

    public Order(long orderId, Customer customer, Product product, LocalDateTime orderDate) {
        this.orderId = orderId;
        this.customer = customer;
        this.product = product;
        this.orderDate = orderDate;
    }
}

@Getter
public class Customer {
    private final long customerId;
    private final String name;
    private final String email;
    private final Address address; // 연관된 객체

    public Customer(long customerId, String name, String email, Address address) {
        this.customerId = customerId;
        this.name = name;
        this.email = email;
        this.address = address;
    }
}

@Getter
public class Product {
    private final long productId;
    private final String productName;
    private final double price;

    public Product(long productId, String productName, double price) {
        this.productId = productId;
        this.productName = productName;
        this.price = price;
    }
}

@Getter
public class Address {
    private final String street;
    private final String city;
    private final String zipCode;

    public Address(String street, String city, String zipCode) {
        this.street = street;
        this.city = city;
        this.zipCode = zipCode;
    }
}
```

### 문제점

- 생성자에 따라서 연관관계가 맺어진 다른 객체들 까지 생성이 필요하고, 또 그 객체에 연결된 객체들 까지 무한히 체이닝될 수 있습니다.

```java
@Test
    void testOrderCreation() {
        // Address 생성
        Address address = new Address("123 Main St", "Seoul", "12345");

        // Customer 생성
        Customer customer = new Customer(1L, 1L, "John Doe", "john@example.com", address);

        // Product 생성
        Product product = new Product(1001L, "Laptop", 1500.00);

        // Order 생성
        Order order = new Order(5001L, customer, product, LocalDateTime.now());

        // 테스트
        assertNotNull(order.getCustomer());
        assertEquals("Laptop", order.getProduct().getProductName());
    }
```

- 위 코드에선 Order 객체를 생성하려면 Customer와 Product 객체가 필요하며, Customer 객체를 생성하려면 Address 객체도 생성해야 합니다.
- 테스트에서 단일 객체 검증을 위해 불필요한 객체들을 생성해야 하는 비효율 발생합니다.

## 3. 객체를 생성할 수 있는 생성자가 존재하지 않는 경우

```java
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Entity
@Table(name = "member")
public class MemberEntity {

    @Id
    private long id;
    private String name;
    private String email;
    private String phoneNumber;
    private int age;
    private boolean isVerified;
    private LocalDateTime createdAt;

    public MemberEntity(String name, String email, String phoneNumber, int age, boolean isVerified) {
        this.name = name;
        this.email = email;
        this.phoneNumber = phoneNumber;
        this.age = age;
        this.isVerified = isVerified;
        this.createdAt = LocalDateTime.now();
    }
}
```

### 문제점

- 기본 생성자 외에, 모든 필드를 초기화하는 생성자를 제공하지 않으므로 테스트에서 객체를 생성할 수 없습니다.
- 테스트를 위해 public 생성자를 추가하거나 setter를 작성해야 하지만, 이는 프로덕션 코드의 불필요한 수정으로 이어집니다.
- 혹은 테스트에서 강제로 객체를 생성하려면 Reflection을 사용해야 할 수 있지만, Reflection은 유지보수성을 떨어뜨리고, 코드 가독성을 저하시킵니다.

## 4. 요약

| 문제 상황 | 설명 | Fixture Monkey 도입 효과 |
| --- | --- | --- |
| 클래스에 final 필드가 포함된 경우 | final 필드로 인해 모든 필드를 초기화해야 객체 생성 가능 | 필요한 데이터만 선택적으로 생성 가능 |
| 객체가 너무 많은 연관 객체 포함 | 생성자 체이닝으로 인해 테스트에 필요한 객체 생성이 복잡해짐 | 의존 객체를 자동 생성하며 필요한 객체만 오버라이드 가능 |
| 생성자가 존재하지 않는 경우 | 생성자가 없어 Reflection 등을 통해 강제로 생성해야 하는 비효율 | 테스트용 생성자 추가 없이 객체 생성 과정 단순화 가능 |

---

# 3. Fixture Monkey의 객체 생성 방식들

`Introspector`란 Fixture Monkey에서 **객체가 생성되는 방법**을 의미합니다.

`FixtureMonkey`가 생성될 때 넣어주는 `Introspector`에 따라 객체가 생성되는 방법이 달라지게 됩니다.

## 1. BeanArbitraryIntrospector

- Fixture Monkey가 객체 생성에 사용하는 기본 introspector 입니다.
- 리플렉션과 setter 메서드를 사용하여 새 인스턴스를 생성하므로 생성할 클래스에는 인자가 없는 생성자(또는 기본생성자)와 setter가 있어합니다.

## 2. ConstructorPropertiesArbitraryIntrospector

- 생성자에 `@ConstructorProperties`가 있거나 없으면 클래스가 레코드 타입이어야 합니다.
(또는 Lombok을 사용하는 경우 lombok.config 파일에 `lombok.anyConstructor.addConstructorProperties=true`를 추가할 수 있습니다.)

## 3. **FieldReflectionArbitraryIntrospector**

- 리플렉션을 사용하여 새 인스턴스를 생성하고 필드를 설정하므로, 생성할 클래스는 인자가 없는 생성자(또는 기본 생성자)와 getter 또는 setter 중 하나를 가져야 합니다.
- 만약 final이 아닌 변수가 선언되어 있다면 getter 또는 setter 없이도 사용 가능합니다.

## 4. BuilderArbitraryIntrospector

- 클래스 빌더를 사용하여 클래스를 생성합니다.

## 5. FailoverArbitraryIntrospector

- 테스트 대상이 너무 다양한 객체를 내부적으로 가지고 있을 때 여러 `Introspector`를 제공할 수 있습니다.
- `FailoverArbitraryIntrospector`를 사용하면 두 개 이상의 introspector를 사용할 수 있으며, introspector 중 하나가 생성에 실패하더라도 `FailoverArbitraryIntrospector`는 계속 다음 introspector로 객체 생성을 시도합니다.

## 6. **PriorityConstructorArbitraryIntrospector**

- 생성자를 사용해서 타입을 생성합니다.
- 다만, 생성자 파라미터 이름을 인식하지 못하면 ArbitraryBuilder API를 사용해 생성자 파라미터를 제어할 수 없습니다.
- 일반적으로 컴파일 시 바이트 코드에 생성자명과 파라미터명이 코드에서 작상한대로 저장되지 않으므로(arg1, arg2 이런식으로 저장됨) 생성자와 파라미터명을 런타임에 알 수 있도록 매핑 시킬 수 있어야 합니다.
- Fixture monkey 공식문서에선 아래 3가지 방법을 제시하고 있습니다.
  - record 타입
  - JVM 옵션 -parameters 활성화
  - 생성자에 @ConstructorProperties 존재

<details>
<summary>ConstructorPropertiesArbitraryIntrospector와의 차이점</summary>

{% asset_img 1.png %}
</details>

## **7. 요약**

| Introspector | 설명 | 특징 |
| --- | --- | --- |
| BeanArbitraryIntrospector | 기본 introspector로, 인자가 없는 생성자와 setter를 이용해 객체 생성 | 일반적인 객체 생성에 적합 |
| ConstructorPropertiesArbitraryIntrospector | @ConstructorProperties 또는 레코드 타입을 통해 생성자 사용 가능 | 생성자 프로퍼티를 통한 특정 생성 가능 |
| FieldReflectionArbitraryIntrospector | 필드 리플렉션으로 객체 생성 | final이 아닌 필드에 대해 setter 없이도 가능 |
| BuilderArbitraryIntrospector | 빌더 패턴을 사용하여 객체 생성 | 빌더 패턴이 있는 객체에 적합 |
| FailoverArbitraryIntrospector | 여러 introspector 중 사용 가능한 방식으로 객체 생성 시도 | 다양한 객체 생성 조건에 유연하게 대응 |
| PriorityConstructorArbitraryIntrospector | 우선 순위가 높은 생성자를 사용하여 객체 생성 | 생성자 우선 사용 상황에 적합 |

---

# 4. 사용 예시

## **1. PriorityConstructorArbitraryIntrospector 사용**

- PriorityConstructorArbitraryIntrospector를 이용해 기 존재하는 생성자를 활용하여 객체를 생성합니다.
- Repository에서 데이터를 조회할 경우 `@QueryProjection` 을 활용하기 위해 DTO(혹은 VO)에서 생성자를 만들어서 사용하고 있는데, 이런 케이스처럼 final 필드가 다수 존재하여, 테스트 객체 생성에 비효율이 발생할 경우 사용하기 좋습니다.
- 해당 예시에선 withParameterNamesResolver() 메서드를 이용하여 생성자의 파라미터명을 명시적으로 작성하였습니다. 혹은 위에 설명한 다른 방법을 이용해 **런타임**에 파라미터명을 알 수 있도록 합니다.

```java
private FixtureMonkey fixtureMonkey;

@BeforeEach
void setUp() {
    this.fixtureMonkey = FixtureMonkey.builder()
            .objectIntrospector(PriorityConstructorArbitraryIntrospector.INSTANCE
                    .withParameterNamesResolver(constructor -> List.of("id", "name", "email", "phoneNumber", "age", "verified", "createdAt")))
            .build();
}

@ParameterizedTest(name = "{index} => age={0}, expectedCategory={1}")
@MethodSource("getPersonVoArguments")
void testPersonVoMethods(Integer age, String expectedCategory) {
    // given
    PersonVo personVo = fixtureMonkey.giveMeBuilder(PersonVo.class)
            .set(javaGetter(PersonVo::getId), 1L)
            .set("age", age)
            .sample();

    // when
    String ageCategory = personVo.getAgeCategory();

    // then
    assertThat(ageCategory).isEqualTo(expectedCategory);
}

private static Stream<Arguments> getPersonVoArguments() {
    return Stream.of(
            Arguments.of(null, "Unknown"),
            Arguments.of(15, "청소년"),
            Arguments.of(30, "청년"),
            Arguments.of(70, "시니어")
    );
}
```

### 실행 결과 

{% asset_img 2.png %}

- 테스트에 필요한 `age`만 넣고 실행 시, 나머지 final 필드들은 알아서 fixture monkey에 의해 채워지거나 null 허용필드인 경우 null이 입력된 것을 볼 수 있습니다.

## 2. **FieldReflectionArbitraryIntrospector 사용**

- 생성자의 파라미터에 존재하지 않는 값 ( id )을 수정하기 위해 `FieldReflectionArbitraryIntrospector` 를 활용하여 객체를 생성합니다.
- Entity class인 경우 보통 생성자에 id 필드를 포함한 생성자가 존재하지 않기 때문에(JPA ID를 생성, 관리 하도록 일임하기 때문) Entity 객체 생성 시 사용하기 좋습니다.

<br />

### Member Entity
```java
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Entity
@Table(name = "member")
public class MemberEntity {

    @Id
    private long id;
    private String name;
    private String email;
    private String phoneNumber;
    private int age;
    private boolean isVerified;
    private LocalDateTime createdAt;

    public MemberEntity(String name, String email, String phoneNumber, int age, boolean isVerified) {
        this.name = name;
        this.email = email;
        this.phoneNumber = phoneNumber;
        this.age = age;
        this.isVerified = isVerified;
        this.createdAt = LocalDateTime.now();
    }

    public void verify() {
        this.isVerified = true;
    }
}
```


### 비즈니스 로직
```java
/**
 * 특정 ID를 가진 회원을 활성화
 * @param memberId 회원 ID
 * @return 활성화된 회원
 */
@Transactional(rollbackFor = Exception.class)
public Member activateMember(long memberId) {
    MemberEntity entity = memberRepository.findById(memberId)
            .orElseThrow(() -> new IllegalArgumentException("존재하지 않는 회원 ID: " + memberId));

    if (entity.isVerified()) {
        throw new IllegalStateException("이미 활성화된 회원입니다: " + memberId);
    }

    entity.verify();
    entity = memberRepository.save(entity);
    return Member.of(entity);
}
```
<br />

### 테스트코드 with. fixture monkey

```java
private FixtureMonkey fixtureMonkey;
private MemberService memberService;

@BeforeEach
void setUp() {
    fixtureMonkey = FixtureMonkey.builder()
            .objectIntrospector(FieldReflectionArbitraryIntrospector.INSTANCE)
            .build();

    memberService = new MemberService(new FakeMemberRepository());
}

@RepeatedTest(value = 100)
void testActivateMemberUsingFixtureMonkey() {
    // given
    MemberEntity entity = fixtureMonkey.giveMeBuilder(MemberEntity.class)
            .set("isVerified", false)
            .sample();

    Member savedMember = memberService.save(entity);

    // when
    Member activatedMember = memberService.activateMember(savedMember.getId());

    // then
    assertThat(activatedMember.isVerified()).isTrue();
    assertThat(entity.getId()).isEqualTo(activatedMember.getId());
}
```

<br />

- fixture monkey를 이용해 MemberEntity 객체 생성 시, 테스트해야하는 비즈니스 로직에서 필요한 값만 지정하여 객체를 생성합니다.
  - 예제에선 isVerified의 상태를 true로 변경하는 것
- 이로 인해 테스트코드의 본래 목적인, 비즈니스 로직 검증에만 집중할 수 있는 간단한 테스트코드를 작성 할 수 있습니다.

---

# 5. 퀴즈

```java
@Component
@RequiredArgsConstructor
public class EventEligibilityChecker {

    private final MemberService memberService;

    @Value("${event.vip.member.ids}")
    private String vipMemberIds;

    private static final ZoneOffset KST = ZoneOffset.of("+09:00");

    /**
     * 특정 이벤트 참여 가능 여부를 확인
     *
     * 조건:
     * - VIP 회원이거나
     * - 관리자 권한을 가진 회원이거나
     * - 이벤트 기간 외에 요청한 회원이라면 false
     *
     * 그렇지 않다면 true를 반환한다.
     *
     * @param memberId 확인할 회원 ID
     * @param eventStartTime 이벤트 시작 시간
     * @param eventEndTime 이벤트 종료 시간
     * @return 참여 가능 여부
     */
    public boolean canParticipate(long memberId, LocalDateTime eventStartTime, LocalDateTime eventEndTime) {
        OffsetDateTime currentTime = OffsetDateTime.now();
        boolean isEligible = true;

        boolean isVipMember = checkVipMember(memberId);
        boolean isAdminMember =
                checkAdminMember(memberId);
        boolean isOutsideEventPeriod = checkOutsideEventPeriod(currentTime, eventStartTime, eventEndTime);

        if (isVipMember || isAdminMember || isOutsideEventPeriod) {
            isEligible = false;
        }

        return isEligible;
    }

    private boolean checkVipMember(long memberId) {
        return Arrays.stream(vipMemberIds.split(","))
                .map(String::trim)
                .map(Integer::parseInt)
                .anyMatch(id -> id == memberId);
    }

    private boolean checkAdminMember(long memberId) {
        return memberService.hasAdminRole(memberId);
    }

    private boolean checkOutsideEventPeriod(OffsetDateTime currentTime, LocalDateTime eventStartTime, LocalDateTime eventEndTime) {
        OffsetDateTime eventStart = eventStartTime.atOffset(KST);
        OffsetDateTime eventEnd = eventEndTime.atOffset(KST);
        return currentTime.isBefore(eventStart) || currentTime.isAfter(eventEnd);
    }
}
```
<br />

위 같은 코드는 어떻게 테스트 코드를 작성 할 수 있을까요?

## 1. 테스트코드 작성

### 1. Fixture Monkey 사용

<details>
<summary>예시 코드</summary>

- 해당 케이스에서 적용 할 수 있는 `Introspector` 가 존재하지 않기 때문에 Fixture monkey로 객체를 생성 할 수 없습니다.
  - 혹 제가 잘못 알고 있다면, 댓글 등으로 조언 부탁드립니다. (__)
</details>

<br />

### 2. @SpringBootTest와 @TestPropertySource 사용


<details>
<summary>예시 코드</summary>

```java
@SpringBootTest
@TestPropertySource(properties = {
        "event.vip.member.ids=1,2,3"
})
class EventEligibilityCheckerTest {

    @Autowired
    private EventEligibilityChecker eventEligibilityChecker;

    @Autowired
    private MemberService memberService;

    @ParameterizedTest(name = "{index} => memberId={0}, startTime={1}, endTime={2}, expectedResult={3}")
    @MethodSource("getMemberArguments")
    void testCanParticipate(int memberId, LocalDateTime startTime, LocalDateTime endTime, boolean expectedResult) {
        // when
        boolean result = eventEligibilityChecker.canParticipate(memberId, startTime, endTime);

        // then
        assertThat(result).isEqualTo(expectedResult);
    }

    static Stream<Object[]> getMemberArguments() {
        return Stream.of(
                new Object[]{1, LocalDateTime.now().minusDays(1), LocalDateTime.now().plusDays(1), false}, // VIP 멤버
                new Object[]{99, LocalDateTime.now().minusDays(1), LocalDateTime.now().plusDays(1), false}, // 관리자 멤버
                new Object[]{4, LocalDateTime.now().plusDays(1), LocalDateTime.now().plusDays(2), false}, // 이벤트 기간 외
                new Object[]{101, LocalDateTime.now().minusDays(1), LocalDateTime.now().plusDays(1), true} // 일반 멤버 (참여 가능)
        );
    }
}
```
</details>

<br />

### 3. Reflection 사용

<details>
<summary>예시 코드</summary>

```java
public class EventEligibilityCheckerTestWithReflection {

    private EventEligibilityChecker eventEligibilityChecker;

    @BeforeEach
    void setUp() {
        eventEligibilityChecker = new EventEligibilityChecker(new MemberService(new FakeMemberRepository()));

        // ReflectionTestUtils를 사용해 @Value 필드 설정
        ReflectionTestUtils.setField(eventEligibilityChecker, "vipMemberIds", "1,2,3");
    }

    @ParameterizedTest(name = "{index} => memberId={0}, startTime={1}, endTime={2}, expectedResult={3}")
    @MethodSource("getMemberArguments")
    void testCanParticipate(int memberId, LocalDateTime startTime, LocalDateTime endTime, boolean expectedResult) {
        // when
        boolean result = eventEligibilityChecker.canParticipate(memberId, startTime, endTime);

        // then
        assertThat(result).isEqualTo(expectedResult);
    }

    static Stream<Object[]> getMemberArguments() {
        return Stream.of(
                new Object[]{1, LocalDateTime.now().minusDays(1), LocalDateTime.now().plusDays(1), false}, // VIP 멤버
                new Object[]{99, LocalDateTime.now().minusDays(1), LocalDateTime.now().plusDays(1), false}, // 관리자 멤버
                new Object[]{4, LocalDateTime.now().plusDays(1), LocalDateTime.now().plusDays(2), false}, // 이벤트 기간 외
                new Object[]{101, LocalDateTime.now().minusDays(1), LocalDateTime.now().plusDays(1), true} // 일반 멤버 (참여 가능)
        );
    }
}
```
</details>

---

# 6. 테스트 하기 쉬운 코드

> No Silver Bullet. - Fred Brooks
> 

단지 테스트코드 작성을 도와주는 도구일 뿐 Fixture Monkey 도 만능은 아닙니다.

테스트코드를 작성하기 어렵다는 건 테스트 하고자 하는 코드 ([System under test](https://en.wikipedia.org/wiki/System_under_test))의 설계 혹은 구조가 잘못되어 있다고 테스트코드가 신호를 보내고 있다고 생각해야합니다.

## 1. 테스트가 어려운 이유

### a. 불확실성

- 불확실성이란 코드가 외부 데이터나 상태에 의존하여 매번 다른 결과를 만들어내는 것을 의미합니다.
- 예를 들어, 현재 시간을 기준으로 동작하는 코드, 랜덤 값을 생성하는 코드, 전역 변수를 사용하는 코드 등은 매 실행 시마다 결과가 달라질 수 있습니다.
- 문제코드에선 `@Value`로 properties에서 주입받는 값들과 `OffsetDateTime.now()`를 불확실성이라고 할 수 있습니다.

### b. 부수효과

- 부수효과라 함은 함수 외부의 상태를 변경하는 것을 의미합니다.
- 예를 들어 메일을 전송하거나 파일에 데이터를 기록하거나 데이터베이스에 저장하는 코드는 외부 시스템에 변화를 일으킵니다.
- 이런 코드들은 단순히 리턴 값을 확인하는 것만으로는 테스트가 불가능하고, 실제로 외부 상태가 바뀌었는지 확인해야 하기 때문에 테스트 비용이 증가하게 됩니다.

## 2. 테스트 하기 쉬운 코드로 수정하기

<details>
<summary>수정된 프로덕션 코드 ( SUT )</summary>
    
```java
@Component
@RequiredArgsConstructor
public class EventEligibilityCheckerV2 {

    private final MemberService memberService;

    private final VIPProperties vipProperties;
    private final ClockProvider clockProvider;

    private static final ZoneOffset KST = ZoneOffset.of("+09:00");

    /**
     * 특정 이벤트 참여 가능 여부를 확인
     *
     * 조건:
     * - VIP 회원이거나
     * - 관리자 권한을 가진 회원이거나
     * - 이벤트 기간 외에 요청한 회원이라면 false
     *
     * 그렇지 않다면 true를 반환한다.
     *
     * @param memberId 확인할 회원 ID
     * @param eventStartTime 이벤트 시작 시간
     * @param eventEndTime 이벤트 종료 시간
     * @return 참여 가능 여부
     */
    public boolean canParticipate(long memberId, LocalDateTime eventStartTime, LocalDateTime eventEndTime) {
        OffsetDateTime currentTime = clockProvider.now();
        boolean isEligible = true;

        boolean isVipMember = checkVipMember(memberId);
        boolean isAdminMember =
                checkAdminMember(memberId);
        boolean isOutsideEventPeriod = checkOutsideEventPeriod(currentTime, eventStartTime, eventEndTime);

        if (isVipMember || isAdminMember || isOutsideEventPeriod) {
            isEligible = false;
        }

        return isEligible;
    }

    private boolean checkVipMember(long memberId) {
        return vipProperties.getIds().contains(memberId);
    }

    private boolean checkAdminMember(long memberId) {
        return memberService.hasAdminRole(memberId);
    }

    private boolean checkOutsideEventPeriod(OffsetDateTime currentTime, LocalDateTime eventStartTime, LocalDateTime eventEndTime) {
        OffsetDateTime eventStart = eventStartTime.atOffset(KST);
        OffsetDateTime eventEnd = eventEndTime.atOffset(KST);
        return currentTime.isBefore(eventStart) || currentTime.isAfter(eventEnd);
    }
}
```

```java
public interface VIPProperties {

    List<Long> getIds();
}

@ConfigurationProperties(prefix = "event.vip.member")
class VIPPropertiesImpl implements VIPProperties {

    private final String ids;

    @ConstructorBinding
    public VIPPropertiesImpl(String ids) {
        this.ids = ids;
    }

    @Override
    public List<Long> getIds() {
        return Arrays.stream(this.ids.split(","))
                .map(String::trim)
                .map(Long::parseLong)
                .toList();
    }
}
...
```
</details>

<br />
<details>
<summary>수정된 테스트 코드</summary>
    
```java
class EventEligibilityCheckerV2Test {

    private EventEligibilityCheckerV2 eventEligibilityChecker;

    @BeforeEach
    void setUp() {
        int year = 2024;
        int month = 12;
        int day =13;
        int hour = 11;
        int minute = 11;
        int second = 11;

        eventEligibilityChecker = new EventEligibilityCheckerV2(
                new MemberService(new FakeMemberRepository())
                , new FakeVIPProperties(Arrays.asList(1L,2L,3L))
                , new FakeSystemClockProvider(OffsetDateTime.of(year, month, day, hour, minute, second, 11, ZoneOffset.UTC))
        );
    }

    @ParameterizedTest(name = "{index} => memberId={0}, startTime={1}, endTime={2}, expectedResult={3}")
    @MethodSource("getMemberArguments")
    void testCanParticipate(int memberId, LocalDateTime startTime, LocalDateTime endTime, boolean expectedResult) {
        // when
        boolean result = eventEligibilityChecker.canParticipate(memberId, startTime, endTime);

        // then
        assertThat(result).isEqualTo(expectedResult);
    }

    static Stream<Object[]> getMemberArguments() {
        int year = 2024;
        int month = 12;
        int day =13;
        int hour = 11;
        int minute = 11;
        int second = 11;

        LocalDateTime eventStartDate = LocalDateTime.of(year, month, day-1, hour, minute, second);
        LocalDateTime eventEndDate = LocalDateTime.of(year, month, day+1, hour, minute, second);

        return Stream.of(
                new Object[]{1, eventStartDate, eventEndDate, false}, // VIP 멤버
                new Object[]{99, eventStartDate, eventEndDate, false}, // 관리자 멤버
                new Object[]{4, eventEndDate.plusDays(1), eventEndDate.plusDays(2), false}, // 이벤트 기간 외
                new Object[]{101, eventStartDate, eventEndDate, true} // 일반 멤버 (참여 가능)
        );
    }
}
```

</details>

<br />

## 3. 코드 개선으로 인해 얻게된 이점
    
1. **책임 분리**: 설정값을 전용 프로퍼티 클래스 (VIPProperties)로 관리하게 되어 EventEligibilityCheckerV2의 책임이 명확해졌습니다.
2. **유연성 증가**: VIPProperties, ClockProvider와 같은 인터페이스를 사용하여 설정값에 대한 종속성이 외부로 분리하였고, 설정값을 추상화하여, 다양한 설정 구현체를 주입할 수 있습니다. 위 예시에선 테스트코드 내에서 Fake 구현체 작성하고 해당 구현체로 객체를 생성 하도록하여 테스트하였습니다.
3. **코드 가독성**: @Value 어노테이션 없이 구성 클래스를 사용하므로, 전체적인 코드의 가독성이 향상됩니다.

---

# 7. 결론

앞서 설명한 설계와 테스트 코드 개선에 더해 Fixture Monkey와 같은 도구를 활용하면 다음과 같은 이점을 얻을 수 있습니다.

## 1. 테스트 데이터 자동 생성

- Fixture Monkey는 복잡한 객체 구조에 대해 필요한 데이터를 자동으로 생성합니다. 이를 통해 테스트를 작성할 때 불필요한 데이터 생성 로직을 줄이고 테스트 로직에 집중할 수 있습니다.
  
## 2. 객체의 불변성과 유연성 유지

- 기존 객체를 수정하거나 Setter를 추가하지 않아도 Fixture Monkey를 통해 객체를 유연하게 생성 및 수정할 수 있습니다. 특히, 불변 객체(immutable object)나 생성자가 많은 객체를 쉽게 테스트할 수 있습니다.

## 3. 데이터 커스터마이징

- 특정 필드에 대한 값을 명시적으로 설정하거나 조건에 맞는 객체를 생성할 수 있습니다. 예를 들어, VIP 멤버 ID 목록처럼 특정 조건을 가진 데이터를 간단하게 생성할 수 있습니다.

## 4. 테스트 유지보수 비용 절감

- 새로운 필드가 추가되거나 객체 구조가 변경되어도 Fixture Monkey를 사용하면 테스트 코드 수정 범위를 최소화할 수 있습니다.

<br />

---

잘 작성된 테스트는 단순히 코드의 동작을 검증하는 역할을 넘어, 코드의 설계와 구조적 결함을 발견하게 해줍니다.  
**Fixture Monkey**와 같은 도구를 활용하면 코드의 불확실성과 부수효과를 줄이면서도 객체 생성과 테스트를 효율적으로 수행할 수 있습니다.  
특히, 복잡한 객체나 다양한 조건의 테스트 데이터를 쉽게 생성할 수 있어 **테스트 작성 비용을 절감**하고 **테스트 품질을 향상**시킬 수 있습니다.  

그러나 **도구의 활용보다 중요한 것은 테스트 가능한 설계를 만드는 것**입니다.  
테스트 가능한 설계는 테스트 코드로 하여금 코드 설계의 개선 방향을 제시하게 하며, 결과적으로 유지보수성과 확장성을 높이는 기반이 됩니다.

긴 글 읽어주셔서 감사합니다.

---

# 참고자료
- [Fixture Monkey 공식문서](https://naver.github.io/fixture-monkey/v1-0-0/)
- [테스트 객체는 엣지 케이스까지 찾아주는 Fixture Monkey에게 맡기세요](https://tv.naver.com/v/23650158)
- [Testable Code by Jin-Wook Chung](https://jwchung.github.io/testable-code)
- [무엇을 테스트할 것인가? 어떻게 테스트할 것인가? (권용근)](https://youtu.be/YdtknE_yPk4?si=d8KPekxKZrsj_Rpk)
- [실무에서 적용하는 테스트 코드 작성 방법과 노하우 (김남윤)](https://www.youtube.com/watch?v=XSkz0kO7J3w)
- [왜 나는 테스트를 작성하기 싫을까? (조성아)](https://www.youtube.com/watch?v=pCE7ibRCZEI)