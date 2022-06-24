---
layout: posts
title: Jackson @JsonCreator를 이용한 역직렬화
date: 2022-04-30 22:00:35
tags:
    - spring
    - jackson
---

## 서론

어쩌다 보니 연속해서 @RequestBody에 관한 글을 작성하게 되었는데... 이전에 말했던 것 처럼 @RequestBody는 Jackson 라이브러리를 사용하기 때문에 기본생성자를 이용해 object(DTO)를 바인딩한다. 그 때문에 다른 생성자에서 validtion등을 처리했다고 해도 당연히 처리되지 않는데 이 때 사용할 수 있는 @JsonCreator에 대해 알아본다.

* * *

## 본론

### 현재 상황

{% asset_img 1.JPG Jackson @JsonCreator를 이용한 역직렬화 %}

<br />

{% asset_img 2.JPG Jackson @JsonCreator를 이용한 역직렬화 %}

<br />

위 이미지들 처럼 Controller와 dto를 만들었다고 했을 때, /test 를 호출해보면 어떻게 될까?

{% asset_img 3.JPG Jackson @JsonCreator를 이용한 역직렬화 %}

<br />

@RequestBody를 사용 할 경우 Jackson 라이브러리를 사용해서 기본 생성자를 이용해 매핑시키기 때문에 "여기타고 생성이 될까요?" 라는 로그는 찍히지 않는다.

이 때, DTO에 처리 혹은 [Assert](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/util/Assert.html) 등을 이용한 validtion을 추가하고 싶다면 어떻게 될까?

생성자에 추가해서 처리하면 좋겠지만, 기본 생성자를 이용해서 생성되므로, controller 혹은 service 단에서 처리해야 한다.(물론 validtion의 경우엔 [@Valid](https://docs.oracle.com/javaee/7/api/javax/validation/Valid.html)를 통해 어느정도 처리 가능하다.)

이런 경우, 기본 생성자가 아닌 다른 생성자를 이용해 dto를 바인딩 시키고 싶은 경우 사용 할 수 있는 annotation이 바로 [@JsonCreator](https://fasterxml.github.io/jackson-annotations/javadoc/2.6/com/fasterxml/jackson/annotation/JsonCreator.html)이다.

이제 다시 코드로 돌아와서, 이 @JsonCreator를 생성자에 선언 해놓고 다시 /test를 요청한다면

{% asset_img 4.JPG Jackson @JsonCreator를 이용한 역직렬화 %}

<br />

아까와 다르게 @Builder 패턴이 적용된 생성자를 통해 바인딩 되어서 "여기타고 생성이 될까요?" 라는 로그가 정상적으로 찍힌 걸 볼 수 있다.

* * *

## 결론

이제 우리는 @RequestBody를 사용할 때 기본 생성자만이 아닌 다른 생성자로도 DTO를 바인딩 할 수 있게 되었으므로, @Valid보다 더 정교한 validtion이나, String을 encoding 해야 할 경우에도 service나 controller가 아닌 생성자에서 처리 할 수 있게 되었으므로 테스트도 더 간편하게 할 수 있게 되었다.

번외로 보통 @JsonCreator는 인스턴스화 할 때 사용 할 생성자를 정의 할 때 사용 하는 annotation으로,
보통 아래 코드처럼 @JsonProperty와 같이 사용해서 엔티티와 완벽히 일치되지 않는 JSON을 역직렬화할 때 주로 사용하는 것 같다.

```java
public class BeanWithCreator {
    public int id;
    public String name;

    @JsonCreator
    public BeanWithCreator(
      @JsonProperty("id") int id, 
      @JsonProperty("theName") String name) {
        this.id = id;
        this.name = name;
    }
}
```

개인적으로 이렇게 쓸 일이 몇 번이나 있을까 싶긴하지만...

* * *

## 전체 코드

- [https://github.com/jungguji/blog-example-code](https://github.com/jungguji/blog-example-code)

## 참고 사이트

- [https://www.baeldung.com/jackson-annotations](https://www.baeldung.com/jackson-annotations)
- [https://fasterxml.github.io/jackson-annotations/javadoc/2.6/com/fasterxml/jackson/annotation/JsonCreator.html](https://fasterxml.github.io/jackson-annotations/javadoc/2.6/com/fasterxml/jackson/annotation/JsonCreator.html)
- 