---
layout: posts
title: 테스트 코드 작성을 더 쉽게 (with. Fixture Monkey)
date: 2024-11-19 23:16:40
tags:
    - test
---

# 1. **Fixture Monkey란?**

Fixture Monkey는 네이버의 내부 프로젝트인 **Plasma**에서 복잡한 테스트 요구 사항을 해결하기 위해 개발된 도구로, 현재는 오픈 소스로 제공되어 Java와 Kotlin 환경에서 쉽게 적용할 수 있습니다. 이 라이브러리는 기존의 고정된 테스트 데이터 대신  복잡한 객체 타입의 테스트 데이터를 자동으로 생성하여 테스트 코드의 일관성과 효율성을 대폭 향상시킬 수 있는 도구입니다.

# 2. Fixture Monkey 도입의 필요성

## 1. 클래스에 final 필드가 포함된 경우

```java
public class PersonVo {
    private final long id;
    private final String name;
    private final String email;
    private final String phoneNumber;
    private final Integer age;
    private final Boolean isVerified;
    private final LocalDateTime createdAt;

    public PersonVo(long id, String name, String email, String phoneNumber, Integer age, Boolean isVerified, LocalDateTime createdAt) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.phoneNumber = phoneNumber;
        this.age = age;
        this.isVerified = isVerified;
        this.createdAt = createdAt;
    }

    public String getAgeCategory() {
        if (this.age == null) {
            return "Unknown";
        } else if (this.age < 18) {
            return "청소년";
        } else if (this.age < 65) {
            return "청년";
        } else {
            return "시니어";
        }
    }

    public String getFormattedCreatedAt() {
        return this.createdAt != null ? this.createdAt.toLocalDate().toString() : "Not Available";
    }
}
```

- 문제점 <br />
PersonVo 클래스의 필드들이 모두 final로 선언되어 있기 때문에, 단순히 테스트에서 특정 메서드(getAgeCategory나 getFormattedCreatedAt)만 검증하려 해도 불필요한 모든 데이터를 포함하여 객체를 생성해야 하는 문제가 있습니다.

```java
class PersonVoTest {

    @Test
    void testGetAgeCategory() {
        // 청소년
        PersonVo minor = new PersonVo(1L, "Alice", "alice@example.com", "123456789", 15, true, LocalDateTime.now());
        assertEquals("청소년", minor.getAgeCategory());

        // 청년
        PersonVo adult = new PersonVo(2L, "Bob", "bob@example.com", "987654321", 30, true, LocalDateTime.now());
        assertEquals("청년", adult.getAgeCategory());

        // 시니어
        PersonVo senior = new PersonVo(3L, "Charlie", "charlie@example.com", "555555555", 70, true, LocalDateTime.now());
        assertEquals("시니어", senior.getAgeCategory());

        // Unknown
        PersonVo unknownAge = new PersonVo(4L, "Diana", "diana@example.com", "111111111", null, true, LocalDateTime.now());
        assertEquals("Unknown", unknownAge.getAgeCategory());
    }

    @Test
    void testGetFormattedCreatedAt() {
        // Valid date
        LocalDateTime validDate = LocalDateTime.of(2024, 11, 19, 10, 0);
        PersonVo personWithValidDate = new PersonVo(1L, "Eve", "eve@example.com", "222222222", 25, true, validDate);
        assertEquals("2024-11-19", personWithValidDate.getFormattedCreatedAt());

        // Not Available
        PersonVo personWithoutDate = new PersonVo(2L, "Frank", "frank@example.com", "333333333", 40, true, null);
        assertEquals("Not Available", personWithoutDate.getFormattedCreatedAt());
    }
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
    Customer customer = new Customer(1L, "John Doe", "john@example.com", address);

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
@Table(name = "post")
public class Post {

    private long id;
    private String title;
    private String content;
    private User owner;
}
```

### 문제점

- 기본 생성자 외에, 모든 필드를 초기화하는 생성자를 제공하지 않으므로 테스트에서 객체를 생성할 수 없습니다.
- 테스트를 위해 public 생성자를 추가하거나 setter를 작성해야 하지만, 이는 프로덕션 코드의 불필요한 수정으로 이어집니다.
- 혹은 테스트에서 강제로 객체를 생성하려면 Reflection을 사용해야 할 수 있지만, Reflection은 유지보수성을 떨어뜨리고, 코드 가독성을 저하시킵니다.

## 2-3. 요약

| 문제 상황 | 설명 | Fixture Monkey 도입 효과 |
| --- | --- | --- |
| 클래스에 final 필드가 포함된 경우 | final 필드로 인해 모든 필드를 초기화해야 객체 생성 가능 | 필요한 데이터만 선택적으로 생성 가능 |
| 객체가 너무 많은 연관 객체 포함 | 생성자 체이닝으로 인해 테스트에 필요한 객체 생성이 복잡해짐 | 의존 객체를 자동 생성하며 필요한 객체만 오버라이드 가능 |
| 생성자가 존재하지 않는 경우 | 생성자가 없어 Reflection 등을 통해 강제로 생성해야 하는 비효율 | 테스트용 생성자 추가 없이 객체 생성 과정 단순화 가능 |

---

## 3. Fixture Monkey의 객체 생성 방식들

`Introspector`란 Fixture Monkey에서 **객체가 생성되는 방법**을 의미합니다.

`FixtureMonkey`가 생성될 때 넣어주는 `Introspector`에 따라 객체가 생성되는 방법이 달라지게 됩니다.

### 1. BeanArbitraryIntrospector

- Fixture Monkey가 객체 생성에 사용하는 기본 introspector 입니다.
- 리플렉션과 setter 메서드를 사용하여 새 인스턴스를 생성하므로 생성할 클래스에는 인자가 없는 생성자(또는 기본생성자)와 setter가 있어합니다.

### 2. ConstructorPropertiesArbitraryIntrospector

- 생성자에 `@ConstructorProperties`가 있거나 없으면 클래스가 레코드 타입이어야 합니다.
(또는 Lombok을 사용하는 경우 lombok.config 파일에 `lombok.anyConstructor.addConstructorProperties=true`를 추가할 수 있습니다.)

### 3. **FieldReflectionArbitraryIntrospector**

- 리플렉션을 사용하여 새 인스턴스를 생성하고 필드를 설정하므로, 생성할 클래스는 인자가 없는 생성자(또는 기본 생성자)와 getter 또는 setter 중 하나를 가져야 합니다.
- 만약 final이 아닌 변수가 선언되어 있다면 getter 또는 setter 없이도 사용 가능합니다.

### 4. BuilderArbitraryIntrospector

- 클래스 빌더를 사용하여 클래스를 생성합니다.

### 5. FailoverArbitraryIntrospector

- 테스트 대상이 너무 다양한 객체를 내부적으로 가지고 있을 때 여러 `Introspector`를 제공할 수 있습니다.
- `FailoverArbitraryIntrospector`를 사용하면 두 개 이상의 introspector를 사용할 수 있으며, introspector 중 하나가 생성에 실패하더라도 `FailoverArbitraryIntrospector`는 계속 다음 introspector로 객체 생성을 시도합니다.

### 6. **PriorityConstructorArbitraryIntrospector**

- 생성자를 사용해서 타입을 생성합니다.
- 다만, 생성자 파라미터 이름을 인식하지 못하면 ArbitraryBuilder API를 사용해 생성자 파라미터를 제어할 수 없습니다.
- 일반적으로 컴파일 시 바이트 코드에 생성자명과 파라미터명이 똑바로 저장되지 않으므로(arg1, arg2 이런식으로 저장됨) 생성자와 파라미터명을 런타임에 알 수 있도록 매핑 시킬 수 있어야 합니다.
- Fixture monkey 공식문서에선 아래 3가지 방법을 제시하고 있습니다.
  - record 타입
  - JVM 옵션 -parameters 활성화
  - 생성자에 @ConstructorProperties 존재

<details>
<summary>ConstructorPropertiesArbitraryIntrospector와의 차이점</summary>

{% asset_img 1.png %}
</details>

## **3-6. 요약**

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

### **1. PriorityConstructorArbitraryIntrospector 사용**

- PriorityConstructorArbitraryIntrospector를 이용해 기 존재하는 생성자를 활용하여 객체를 생성합니다.
- Repository에서 데이터를 조회할 경우 `@QueryProjection` 을 활용하기 위해 DTO(혹은 VO)에서 생성자를 만들어서 사용하고 있는데, 이런 케이스에서 사용하기 좋습니다.

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

### 2. **FieldReflectionArbitraryIntrospector 사용**

- 생성자의 파라미터에 존재하지 않는 값 ( id )을 수정하기 위해 `FieldReflectionArbitraryIntrospector` 를 활용하여 객체를 생성합니다.
- Entity class인 경우 보통 생성자에 id 필드를 포함한 생성자가 존재하지 않기 때문에 Entity 객체 생성 시 사용하기 좋습니다.