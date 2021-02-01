---
layout: posts
title: Junit4 설정 주입 에러
date: 2021-01-12 21:45:32
tags:
    - Junit
    - Test
    - Spring
---

## 서론

현재 작업 중인 프로젝트에서 테스트 코드를 작성해 테스트할 일이 있었는데 프로젝트의 환경은 spring 4.3에 Junit 4.8이었다.

이에 원래 사용하던 junit5로 넘어갈까 하였으나 junit5를 사용하려면 설정을 spring boot으로 해야 한다는 글들이 있어 같은 테스트 환경을 만들기 위해 junit만 4.12 버전으로 업그레이드한 후 테스트를 진행하였는데 spring 프로젝트지만 config 설정들을 boot 처럼 java 파일로 관리하는 형태여서 java 파일과 properties 파일을 동시에 잡아 줄 필요가 있었는데
@Contextconfiguration(classes = {블라블라...}, locations = {블라블라...})로 잡으니 에러가 발생하여 해결한 방법을 작성해놓는다.

* * *

## 본론

### 코드

```java
@ContextConfiguration(classes = {
        DatabaseConfig.class,
        SecurityConfig.class,
        SocialConfig.class,
        EnumConfig.class,
        WebMvcConfig.class
}, locations = "classpath:properties/test.properties")
```

서론에 적은 것 처럼 classes와 locations를 둘 다 설정하였더니 아래와 같이 에러가 발생하였다.

### 에러

```java
java.lang.IllegalArgumentException: Cannot process locations AND classes for context configuration [ContextConfigurationAttributes@64c87930 declaringClass = 'com.xxxx.xxxImplTest', classes = '{class com.xxxx.config.DatabaseConfig, class com.xxx.config.SecurityConfig, class com.xxx.config.SocialConfig, class com.xxx.config.EnumConfig, class com.xxx.config.WebMvcConfig}', locations = '{classpath:properties/test.properties}', inheritLocations = true, initializers = '{}', inheritInitializers = true, name = [null], contextLoaderClass = 'org.springframework.test.context.ContextLoader']: configure one or the other, but not both.

	at org.springframework.util.Assert.isTrue(Assert.java:68)
	at org.springframework.test.context.support.AbstractDelegatingSmartContextLoader.processContextConfiguration(AbstractDelegatingSmartContextLoader.java:154)
	at org.springframework.test.context.support.AbstractTestContextBootstrapper.buildMergedContextConfiguration(AbstractTestContextBootstrapper.java:371)
	at org.springframework.test.context.support.AbstractTestContextBootstrapper.buildMergedContextConfiguration(AbstractTestContextBootstrapper.java:305)
	at org.springframework.test.context.support.AbstractTestContextBootstrapper.buildTestContext(AbstractTestContextBootstrapper.java:112)
	at org.springframework.test.context.TestContextManager.<init>(TestContextManager.java:120)
	at org.springframework.test.context.TestContextManager.<init>(TestContextManager.java:105)
	at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.createTestContextManager(SpringJUnit4ClassRunner.java:152)
	at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.<init>(SpringJUnit4ClassRunner.java:143)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
	at org.junit.internal.builders.AnnotatedBuilder.buildRunner(AnnotatedBuilder.java:104)
	at org.junit.internal.builders.AnnotatedBuilder.runnerForClass(AnnotatedBuilder.java:86)
	at org.junit.runners.model.RunnerBuilder.safeRunnerForClass(RunnerBuilder.java:70)
	at org.junit.internal.builders.AllDefaultPossibilitiesBuilder.runnerForClass(AllDefaultPossibilitiesBuilder.java:37)
	at org.junit.runners.model.RunnerBuilder.safeRunnerForClass(RunnerBuilder.java:70)
	at org.junit.internal.requests.ClassRequest.createRunner(ClassRequest.java:28)
	at org.junit.internal.requests.MemoizingRequest.getRunner(MemoizingRequest.java:19)
	at org.junit.internal.requests.FilterRequest.getRunner(FilterRequest.java:36)
	at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:49)
	at com.intellij.rt.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:33)
	at com.intellij.rt.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:230)
	at com.intellij.rt.junit.JUnitStarter.main(JUnitStarter.java:58)
```

* * *
## 해결

@Testpropertysource(classpath:"")로 test properties를 잡아주고
@Contextconfiguration(classes = { XXXConfig.class}) 로 Java config 파일들을 잡아준다.

```java
@ContextConfiguration(classes = {
        DatabaseConfig.class,
        SecurityConfig.class,
        SocialConfig.class,
        EnumConfig.class,
        WebMvcConfig.class
})
@Testpropertysource("classpath:properties/test.properties")
```