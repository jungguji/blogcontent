---
layout: posts
title: Intelli J 코드 자동생성(with Live Template)
date: 2022-10-16 15:53:21
tags:
    - Intelli J
---

## 서론

 필자는 항상 단순하고 반복적인 일은 기계가 하고, 사람은 사색해야 한다고 생각한다. 당연하지만 이러한 생각은, 코딩할 때도 마찬가지다. class를 생성할 때마다 붙여야 하는 Annotation들을 (예를 들면 @Getter 같은) 붙이는 게 그렇게 귀찮을 수가 없었는데... 그러던 중 Intelli J의 [Live Template](https://blog.jetbrains.com/ko/2020/05/18/write-code-faster-using-live-templates-ko/) 이라는 기능을 발견하여 필자 같은 사람들의 고통을 줄여 주고자 이 글을 작성한다.
* * *

## 본론

{% asset_img "1_settings.PNG" "Perference창" %}

<br />
우선 위 이미지처럼 preference(settings)을 열어서 'Live Templates'를 검색해서 Live Templates 메뉴를 클릭한다.

{% asset_img "2_template_group.png" "Template group 생성" %}

<br />

열면 오른쪽에 여러 Template 그룹들이 존재하는데 필자처럼 Java가 없다면 Java 그룹을 생성해주거나, 기존 그룹에 탬플릿만 추가하여도 된다.

{% asset_img "3_template_생성.PNG" "Template 생성" %}

<br />

필자는 Entity 클래스를 생성하거나, request, response dto를 생성할 때마다 

```java
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
```

annotation을 붙이는게 너무 귀찮아서 위 코드를 탬플릿으로 지정하고 단축키는 'gn'으로 설정해줬다.


{% asset_img "4_template_적용위치.png" "Template 생성" %}

<br />

그리고 해당 annotation들은 class에만 적용하면 되므로 Declaration으로만 지정해주었다.

{% asset_img "5_생성.gif" "탬플릿화 한 코드 생성" %}

<br />

모든 설정을 완료하고 이제 설정한 단축키를 지정한 위치에서 사용해보면 이처럼 간단하게 우리가 탬플릿화한 코드가 자동으로 생성되는 모습을 볼 수 있다.

이 글에선 소개하지 않았지만, template text를 입력하는 곳에서 `$METHOD_NAME$` 처럼 `$$` 으로 변수를 선언하여 사용하는 방법들도 있으니, 필요한 사람들은 꼭 활용하여 생산성을 높일 수 있기를 바란다. 
* * *

## 결론

 이렇게 우리는 또 작은 할 일과 짧은 시간을 기계에 넘기고 다른 생산적인 일을 할 시간을 얻을 수 있었다. 티끌 모아 태산이라는 말이 있듯이 이런 작은 일들이 줄여서 큰일을 할 수 있기를 바라며 글을 마친다.


## 참고 사이트

- [https://velog.io/@max9106/IntelliJ-Live-Template](https://velog.io/@max9106/IntelliJ-Live-Template)


