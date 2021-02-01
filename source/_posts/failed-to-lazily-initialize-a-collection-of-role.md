---
layout: posts
title: failed to lazily initialize a collection of role
date: 2021-01-22 22:31:53
tags:
    - Junit
    - JPA
---

## 서론

연관관계에 있는 객체를 가져와서 set 하는 메서드를 테스트 하는 도중 아래와 같은 에러가 발생하였다.
* * *

## 본론

### 해당 에러

```java
org.hibernate.LazyInitializationException: failed to lazily initialize a collection of role: com.jjlee.wedding.payment.domain.Cost.costOptions, no session or session was closed

	at org.hibernate.collection.AbstractPersistentCollection.throwLazyInitializationException(AbstractPersistentCollection.java:383)
	at org.hibernate.collection.AbstractPersistentCollection.throwLazyInitializationExceptionIfNotConnected(AbstractPersistentCollection.java:375)
	at org.hibernate.collection.AbstractPersistentCollection.readSize(AbstractPersistentCollection.java:122)
	at org.hibernate.collection.PersistentBag.size(PersistentBag.java:248)
	at com.xxxxxx.ImplTest.가져와서_셋해야하는지_테스트(CostServiceImplTest.java:89)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.junit.runners.model.FrameworkMethod$1.runReflectiveCall(FrameworkMethod.java:59)
	at org.junit.internal.runners.model.ReflectiveCallable.run(ReflectiveCallable.java:12)
	at org.junit.runners.model.FrameworkMethod.invokeExplosively(FrameworkMethod.java:56)
	at org.junit.internal.runners.statements.InvokeMethod.evaluate(InvokeMethod.java:17)
	at org.junit.internal.runners.statements.RunBefores.evaluate(RunBefores.java:26)
	at org.springframework.test.context.junit4.statements.RunBeforeTestMethodCallbacks.evaluate(RunBeforeTestMethodCallbacks.java:75)
	at org.springframework.test.context.junit4.statements.RunAfterTestMethodCallbacks.evaluate(RunAfterTestMethodCallbacks.java:86)
	at org.springframework.test.context.junit4.statements.SpringRepeat.evaluate(SpringRepeat.java:84)
	at org.junit.runners.ParentRunner.runLeaf(ParentRunner.java:366)
	at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.runChild(SpringJUnit4ClassRunner.java:252)
	at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.runChild(SpringJUnit4ClassRunner.java:94)
	at org.junit.runners.ParentRunner$4.run(ParentRunner.java:331)
	at org.junit.runners.ParentRunner$1.schedule(ParentRunner.java:79)
	at org.junit.runners.ParentRunner.runChildren(ParentRunner.java:329)
	at org.junit.runners.ParentRunner.access$100(ParentRunner.java:66)
	at org.junit.runners.ParentRunner$2.evaluate(ParentRunner.java:293)
	at org.springframework.test.context.junit4.statements.RunBeforeTestClassCallbacks.evaluate(RunBeforeTestClassCallbacks.java:61)
	at org.springframework.test.context.junit4.statements.RunAfterTestClassCallbacks.evaluate(RunAfterTestClassCallbacks.java:70)
	at org.junit.runners.ParentRunner$3.evaluate(ParentRunner.java:306)
	at org.junit.runners.ParentRunner.run(ParentRunner.java:413)
	at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.run(SpringJUnit4ClassRunner.java:191)
	at org.junit.runner.JUnitCore.run(JUnitCore.java:137)
	at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:68)
	at com.intellij.rt.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:33)
	at com.intellij.rt.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:230)
	at com.intellij.rt.junit.JUnitStarter.main(JUnitStarter.java:58)
```

이 때 디버깅을 해보면 연결된 entitiy에서

```java
Unable to evaluate the expression Method threw 'org.hibernate.LazyInitializationException' exception.
```

라는 에러가 발생해 있는데 이는 결국 select 한 entitiy가 영속성 컨텍스트 내에 존재하지 않기 때문에 발생한 애러이다. 
위 에러에서 말하는 세션이 바로 영속성 컨텍스트를 말하는 것이고, 이는 한 트랜잭션안에 해당 entitiy가 존재하지 않다는 것과 같은 말이다.

### 해결

이를 해결하기 위해선 현재 지연로딩으로 되어있는 연관관계를 즉시로딩으로 변경하여 한번에 가져오던가,
혹은 Test 메서드에 @Transactional 어노테이션을 줘서 트랜잭션 내에 존재하도록 해주면 테스트가 정상적으로 통과되게 된다.

### 결과

{% asset_img 성공.PNG failed to lazily initialize a collection of role "에러 수정 후 성공한 테스트" "에러 수정 후 성공한 테스트" %}
* * *

## 참고 사이트

- [[https://bebong.tistory.com/entry/JPA-Lazy-Evaluation-LazyInitializationException-could-not-initialize-proxy-–-no-Session](https://bebong.tistory.com/entry/JPA-Lazy-Evaluation-LazyInitializationException-could-not-initialize-proxy-%E2%80%93-no-Session)]([https://bebong.tistory.com/entry/JPA-Lazy-Evaluation-LazyInitializationException-could-not-initialize-proxy-–-no-Session](https://bebong.tistory.com/entry/JPA-Lazy-Evaluation-LazyInitializationException-could-not-initialize-proxy-%E2%80%93-no-Session))
- [[https://ankonichijyou.tistory.com/entry/JPA-OneToMany-오류](https://ankonichijyou.tistory.com/entry/JPA-OneToMany-%EC%98%A4%EB%A5%98)]([https://ankonichijyou.tistory.com/entry/JPA-OneToMany-오류](https://ankonichijyou.tistory.com/entry/JPA-OneToMany-%EC%98%A4%EB%A5%98))