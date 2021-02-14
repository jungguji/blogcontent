---
layout: posts
title: Requestbody Annotation 사용 시 기본 생성자 필요
date: 2021-02-05 22:39:20
tags:
    - Junit
    - Test
---

## 서론

사실, 제목이 결론이다. @Requestbody Annotation을 사용하려면 반드시 해당 DTO에는 기본 생성자가 명시적으로 존재하여야 한다. 뭔가 설정을 잘못했는지는 모르겠지만 intellij 에서 stacktrace가 출력되지 않아 이 사실을 알 수 가 없어서 엄청나게 삽질을 했기에 내용을 정리한다...
* * *

## 본론

### 상황

controller 테스트에서 모든 조건을 맞췄는데도 500 에러가 발생하여 테스트를 통과하지 못하는 상황이었는데 @Requestbody annotation을 DTO가 아닌 HashMap으로 변경하니 테스트가 정상적으로 통과되어 JSON이 제대로 DTO로 매핑되지 않아서 발생하는 문제라고 판단했다.

### 에러 발생

{% asset_img 기본생성자_에러.png Requestbody Annotation 사용 시 기본 생성자 필요 "에러 발생" "에러 발생" %}

> No suitable constructor found for type [simple type, class 클래스명]: can not instantiate from JSON object (missing default constructor or creator, or perhaps need to add/enable type information?)

직역하면 기본 생성자가 존재하지 않아서 에러가 발생했다는 내용이다.

### 해결 방법

빌더패턴을 사용하던 클래스에 @Noargsconstructor annotation을 사용해서 기본 생성자를 생성하게 하였다.

{% asset_img 성공.png Requestbody Annotation 사용 시 기본 생성자 필요 "성공" "테스트 성공" %}

## 결론

@Requestbody와 DTO를 매핑되게 해야 할 경우 많은 조건이 필요하다. 
이전 포스트에 작성했던 boolean의 변수명이라던가, 지금과 같은 기본 생성자라 라던지... 이게 모두 @Requestbody가 jackson 라이브러리를 이용해 매핑하기 때문이다. 

* * *

## 참고 사이트

- [https://velog.io/@conatuseus/RequestBody%EC%97%90-%EA%B8%B0%EB%B3%B8-%EC%83%9D%EC%84%B1%EC%9E%90%EB%8A%94-%EC%99%9C-%ED%95%84%EC%9A%94%ED%95%9C%EA%B0%80](https://velog.io/@conatuseus/RequestBody%EC%97%90-%EA%B8%B0%EB%B3%B8-%EC%83%9D%EC%84%B1%EC%9E%90%EB%8A%94-%EC%99%9C-%ED%95%84%EC%9A%94%ED%95%9C%EA%B0%800)
