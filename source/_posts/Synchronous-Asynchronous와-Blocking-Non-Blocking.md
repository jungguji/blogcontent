---
layout: posts
title: 'Synchronous, Asynchronous와 Blocking, Non-Blocking'
date: 2020-11-16 22:47:28
tags:
    - 면접 질문
    - 개발
---

## 결론
* Synchronous, Asynchronous 는 두 개 이상의 대상에 대한 시간을 관리하는 방법이고,
* Blocking, Non-Blocking 은 직접 제어 할 수 없는 대상을 처리하는 방법에 대한 것이다.

### Synchronous, Asynchronous

* 두 개 이상의 대상들의 시간을 처리하는 방식
* Synchronous는 두 개 이상의 대상들의 시간을 일치 시키는 것 으로 메서드가 종료되는 시간과 결과를 전달받은 시간이 일치하면 Synchronous(동기) 방식이다.
* Asynchronous는 두 개 이상의 대상들의 시간을 일치 시키지 않는 것 으로 메서드가 종료되는 시간과 결과를 전달받은 시간이 일치하지 않으면 Asynchronous(비동기) 방식이다.

### Blocking, Non-Blocking

* 직접 제어 할 수 없는 대상에 대한 처리 방식
* Blocking은 직접 제어 할 수 없는 대상이 작업이 끝날 때 까지 __제어권을 넘기지 않아 작업이 끝날 때 까지 대기__ 하는 방식이다.
* Non-Blocking은 직접 제어 할 수 없는 대상의 작업의 종료와 관계없이 __제어권을 바로 다시 얻어와 다른 일을 할 수 있게__ 하는 방식이다.
* * *

## 본론

### Synchronous

우선 Synchronous의 뜻을 사전에선 아래와 같이 정의하고 있다.

{% blockquote 네이버 어학사전 %}
동시 발생하는
{% endblockquote %}

이 처럼 두 개 이상의 작업을 동시에 발생시키거나, 동시에 종료시키는 행동을 말한다.

동기 방식을 사용 할 경우 메서드를 호출 했을 때, 결과를 호출한 쪽에서 스스로 확인하고 처리하기 때문에, 결과를 처리 할 때까지 기다렸다가 결과를 전달받고 메서드를 종료 시킨다.

간단히 요약하면 메서드를 호출하면 그 메서드가 종료되는 시간과 결과가 return 되는 시간이 같다는 것으로, 메서드가 종료되는 시간과 결과가 return 되는 시간이 같으므로 동기 방식이라 말한다.

혹은, 2개 이상의 일을 각각 실행시켜 A가 끝나는 시간과 B가 시작하는 시간이 같은 경우도 마찬가지로 동기 방식이라고 할 수 있다.

Java로 예를 들면 쓰레드를 동기 시키기 위해 사용하는 [synchronized](https://www.javamex.com/tutorials/synchronization_concurrency_synchronized1.shtml)가 있고 일반적으로는 일부러 설정하지 않는 한 보통의 메서드 호출은 모두 동기 방식으로 사용된다.

### Asynchronous

Asynchronous, 비동기 방식은 부정형 접두사 A를 붙여서 두 개 이상의 대상의 시간을 일치 시키지 않는 방식으로 동기 방식과 반대로 동작한다.

즉, 메서드를 호출 했을 때 결과가 return되지 않더라도 다른 작업을 수행하거나 메서드를 종료하고 나중에 처리되면 그 때 결과물을 가져온다.

이 때 메서드를 호출한 시간과 return된 결과를 받은 시간이 일치하지 않으므로 이러한 방식을 비동기 방식이라 한다.

보통 ajax를 통해 많이 사용 해봤을 것이고, Java에선 쓰레드를 활용해 비동기적으로 처리 할 수 있고, Spring에선 [@Async Annotation](https://www.baeldung.com/spring-async)을 이용해 비동기 방식을 처리 할 수 있다.

### Blocking

결론에서 말한 것 처럼 Blocking은 직접 제어 할 수 없는 대상이 작업이 끝날 때 까지 제어권을 넘기지 않아 작업이 끝날 때 까지 대기하게 하는 방식이다.

예를 들면 스타크래프트에서 테란의 SCV는 한번 건물을 짓기 시작하면 건설이 끝날 때 까지 다른 동작을 할 수 없으므로, Blocking 상태라고 할 수 있고, Java에선 System.in 같은 I/O 처리를 예로 들 수 있다.

### Non-Blocking

Blocking과 반대되는 방식으로 작업의 처리 여부와 상관없이 작업을 호출한 쪽이 다시 제어권을 가져와서 자유롭게 다른 일을 처리할 수 있는 방식을 Non-Blocking 방식이라고 한다.

마찬가지의 예로 프로토스의 프로브는 건물을 짓게 하면 차원 균열만 개방하고 바로 미네라를 캐는 등의 다른 일을 수행 할 수 있으므로 프로브는 Non-Blocking 상태라고 할 수 있다.

<br />

{% asset_img scv.gif Synchronous, Asynchronous와 Blocking, Non-Blocking "서플라이디포를 건설 하는 동안 다른 행위를 할 수 없는 SCV" "'서플라이디포를 건설 하는 동안 다른 행위를 할 수 없는 SCV'" %}

<br />

{% asset_img 프로브.png Synchronous, Asynchronous와 Blocking, Non-Blocking "첫 번째 파일럿을 소환 후 파일럿이 완성되기 전에 두 번째 파일럿을 소환 할 수 있는 프로브" "'첫 번째 파일럿을 소환 후 파일럿이 완성되기 전에 두 번째 파일럿을 소환 할 수 있는 프로브'" %}
* * *

## 참고 사이트

* https://velog.io/@codemcd/Sync-VS-Async-Blocking-VS-Non-Blocking-sak6d01fhx
* https://www.youtube.com/watch?v=HKlUvCv9hvA&t=806s
* https://nesoy.github.io/articles/2017-01/Synchronized
* https://homoefficio.github.io/2017/02/19/Blocking-NonBlocking-Synchronous-Asynchronous/
* https://www.notion.so/Sync-vs-Async-Block-vs-Non-block-da3e458e74c84201abfd4bcbf2ed00c2
