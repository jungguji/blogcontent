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

- 문제 설명 <br />
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

