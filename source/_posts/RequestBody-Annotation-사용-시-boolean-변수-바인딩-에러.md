---
layout: posts
title: RequestBody Annotation 사용 시 boolean 변수 바인딩 에러
date: 2020-12-31 22:08:54
tags:
    -Spring-Boot
    -
---

## 서론

토이프로젝트 중 @Reqeust 어노테이션을 적용한 DTO에서 boolean 데이터를 제대로 전달 받지 못하는 문제가 발생하여 이를 정리한다.
* * *

## 문제 발생

vue.js에서 넘어온 데이터를 @RequestBody 어노테이션을 활용해 DTO 객체로 전달 받으려 하였는데 boolean 타입의 데이터가 정상적으로 넘어오지 않는 문제가 발생하였다.

### 문제가 발생한 Test Code

```java
@Test
public void dto_boolean_test() throws Exception {
    //given
    RequestDTO dto = RequestDTO.builder()
            .isTestCase(true)
            .isNQuantity(true)
            .isSpaceIncludeNumber(true)
            .build();

    String test = objectMapper.writeValueAsString(dto);

    System.out.println(test);
    final ResultActions action = mockMvc.perform(post("/frame")
            .contentType(MediaType.APPLICATION_JSON)
            .content(test))
            .andDo(print());

    //then
    MvcResult result = action
            .andExpect(status().isOk())
            .andExpect(content().json(test))
            .andReturn();
}
```

### 에러 결과

{% asset_img test실패.PNG RequestBody Annotation 사용 시 boolean 변수 바인딩 에러 %}
* * * 

## 원인

{% blockquote Project Lombok https://projectlombok.org/features/GetterSetter Project Lombok %}
 You can annotate any field with @Getter and/or @Setter, to let lombok generate the default getter/setter automatically.
A default getter simply returns the field, and is named getFoo if the field is called foo (or isFoo if the field's type is boolean). 
{% endblockquote %}

위 설명처럼 lombok에서 제공하는 @Getter 혹은 @Setter 어노테이션을 사용 할 경우 자동으로 getter/setter 메서드를 생성해주는데
이 때 boolean 타입의 변수에 붙는 prefix는 get이 아닌 is이므로 @RequestBody에서 찾을 수 없어 바인딩 되지 않아 발생하는 문제였다.

### 실패한 코드
{% asset_img 실패한_코드.PNG RequestBody Annotation 사용 시 boolean 변수 바인딩 에러 "실패한_코드 실패했을 당시 DTO" "실패한_코드 실패했을 당시 DTO" %}

이 처럼 boolean 변수에 is prefix를 붙여놓은 상태에서 @Getter 어노테이션을 사용하니, 내부적으로 isIsTestCase() 같은 이상한 네이밍의 메서드가 생성되서
@RequestBody에서 바인딩에 사용하는 Jackson 라이브러리의 ObjectMapper에서 필드를 찾을 수 없어서 바인딩 되지 않아 DTO에 정상적으로 값이 입력되지 않았던 것이다.

{% blockquote Jackson ObjectMapper https://tutorials.jenkov.com/java-json/jackson-objectmapper.html#how-jackson-objectmapper-matches-json-fields-to-java-fields Jackson ObjectMapper %}
By default Jackson maps the fields of a JSON object to fields in a Java object by matching the names of the JSON field to the getter and setter methods in the Java object. Jackson removes the "get" and "set" part of the names of the getter and setter methods, and converts the first character of the remaining name to lowercase.
{% endblockquote %}
* * * 

## 해결

1. boolean 변수명에서 is prefix를 제거한다.
2. default로 false로 되어 있는 lombok.getter.noIsPrefix=true 설정을 추가한다.
이 설정을 추가하면 boolean 변수도 get prefix를 사용한다.

필자는 boolean 변수에서 is prefix를 제거하는 방식으로 처리했다.
다른 타입은 자료형에 따라 prefix를 붙이지 않는 상황에서 boolean 변수에만 붙이는 것이 옳지 않다고 생각했기 때문에 1번을 선택했다.

### 수정된 코드

{% asset_img 성공한_코드.PNG RequestBody Annotation 사용 시 boolean 변수 바인딩 "에러 문제를 수정한 DTO" "에러 문제를 수정한 DTO" %}

### 결과

{% asset_img test성공.PNG RequestBody Annotation 사용 시 boolean 변수 바인딩 "에러 수정 후 성공한 테스트" "에러 수정 후 성공한 테스트" %}
* * *

## 참고 사이트

* [https://projectlombok.org/features/GetterSetter](https://projectlombok.org/features/GetterSetter)
* [https://velog.io/@conatuseus/RequestBody%EC%97%90-%EC%99%9C-%EA%B8%B0%EB%B3%B8-%EC%83%9D%EC%84%B1%EC%9E%90%EB%8A%94-%ED%95%84%EC%9A%94%ED%95%98%EA%B3%A0-Setter%EB%8A%94-%ED%95%84%EC%9A%94-%EC%97%86%EC%9D%84%EA%B9%8C-3-idnrafiw](https://velog.io/@conatuseus/RequestBody%EC%97%90-%EC%99%9C-%EA%B8%B0%EB%B3%B8-%EC%83%9D%EC%84%B1%EC%9E%90%EB%8A%94-%ED%95%84%EC%9A%94%ED%95%98%EA%B3%A0-Setter%EB%8A%94-%ED%95%84%EC%9A%94-%EC%97%86%EC%9D%84%EA%B9%8C-3-idnrafiw)
* [http://tutorials.jenkov.com/java-json/jackson-objectmapper.html#how-jackson-objectmapper-matches-json-fields-to-java-fields](http://tutorials.jenkov.com/java-json/jackson-objectmapper.html#how-jackson-objectmapper-matches-json-fields-to-java-fields)
