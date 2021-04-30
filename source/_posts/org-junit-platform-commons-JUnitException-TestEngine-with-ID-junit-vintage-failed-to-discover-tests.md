---
layout: posts
title: >-
  org.junit.platform.commons.JUnitException: TestEngine with ID junit-vintage
  failed to discover tests
date: 2021-04-19 22:09:46
tags:
  - Junit
  - 개발
  - 오류
---

## 서론

기존에 jar파일을 추가해서 Junit4로 테스트하던 프로젝트를 마찬가지로 jar파일을 Junit5로 버전 업해서 사용하고 있었는데
'굳이 이럴 필요없이 maven 프로젝트로 변경하면 되지 않나?'라는 생각에 maven 프로젝트로 변경 후 발생했던 에러를 해결한 방법을 작성해둔다...

## 상황

위에서 적은 것 처럼 기존에 Junit jar를 다운받아서 직접 경로를 지정하는 방식으로 junit4, 5를 둘 다 사용하고 있었는데 프로젝트를 maven으로 변경 후 의존성 충돌이 발생해 아래와 같은 에러가 발생하며 테스트가 진행되지 않았다.

### 발생한 에러

```java
Internal Error occurred.
org.junit.platform.commons.JUnitException: TestEngine with ID 'junit-vintage' failed to discover tests
  at org.junit.platform.launcher.core.EngineDiscoveryOrchestrator.discoverEngineRoot(EngineDiscoveryOrchestrator.java:111)
  at org.junit.platform.launcher.core.EngineDiscoveryOrchestrator.discover(EngineDiscoveryOrchestrator.java:85)
  at org.junit.platform.launcher.core.DefaultLauncher.discover(DefaultLauncher.java:92)
  at org.junit.platform.launcher.core.DefaultLauncher.execute(DefaultLauncher.java:75)
  at com.intellij.junit5.JUnit5IdeaTestRunner.startRunnerWithArgs(JUnit5IdeaTestRunner.java:71)
  at com.intellij.rt.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:33)
  at com.intellij.rt.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:220)
  at com.intellij.rt.junit.JUnitStarter.main(JUnitStarter.java:53)
Caused by: org.junit.platform.commons.JUnitException: Unsupported version of junit:junit: 3.8.1. Please upgrade to version 4.12 or later.
  at org.junit.vintage.engine.JUnit4VersionCheck.checkSupported(JUnit4VersionCheck.java:49)
  at org.junit.vintage.engine.JUnit4VersionCheck.checkSupported(JUnit4VersionCheck.java:35)
  at org.junit.vintage.engine.VintageTestEngine.discover(VintageTestEngine.java:62)
  at org.junit.platform.launcher.core.EngineDiscoveryOrchestrator.discoverEngineRoot(EngineDiscoveryOrchestrator.java:103)
  ... 7 more

Process finished with exit code -2
```

위 에러 로그처럼 'junit-vintage'를 아이디로 하는 테스트 엔진을 찾지 못했다는 에러가 발생해서 [junit-vintage](https://mvnrepository.com/artifact/org.junit.vintage/junit-vintage-engine) 의존성을 추가해주었는데..
* * *

## 그 후 삽질

```xml
<!-- https://mvnrepository.com/artifact/org.junit.vintage/junit-vintage-engine -->
<dependency>
    <groupId>org.junit.vintage</groupId>
    <artifactId>junit-vintage-engine</artifactId>
    <version>5.7.1</version>
    <scope>test</scope>
</dependency>
```

추가 해줘도 똑같은 에러만 반복 할 뿐이었다.

그래서 공식 문서를 살펴보니..

{% blockquote JUnit 5 User Guide https://junit.org/junit5/docs/current/user-guide/#overview-what-is-junit-5 What is JUnit 5? %}
JUnit 5 = JUnit Platform + JUnit Jupiter + JUnit Vintage
{% endblockquote %}

라고 떡하니 써 적혀있어서 위에 3 개의 의존성을 모두 추가한 후 리빌드 후 다시 테스트를 실행해봤다.

```xml
<!-- Only needed to run tests in a version of IntelliJ IDEA that bundles older versions -->
<dependency>
    <groupId>org.junit.platform</groupId>
    <artifactId>junit-platform-launcher</artifactId>
    <version>1.7.1</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.7.1</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.junit.vintage</groupId>
    <artifactId>junit-vintage-engine</artifactId>
    <version>5.7.1</version>
    <scope>test</scope>
</dependency>
```

하지만 결과는 마찬가지였고... .m2폴더에서 Junit 의존성을 다 삭제하고, Intellij project setting에서 Modules, Libraies에서 모두 삭제하고 이것저것 다 해봤지만 모두 실패로 돌아가서 결국 새 maven 프로젝트를 만들어서 하나씩 비교해보기로 했다.
* * *

## 해결

그 결과... .iml 파일에 아래 처럼 의존성들이 남아있는 것을 확인했고...

```xml
<orderEntry type="module-library" scope="TEST">
  <library name="JUnit5.4">
    <CLASSES>
      <root url="jar://$MAVEN_REPOSITORY$/org/junit/jupiter/junit-jupiter/5.4.2/junit-jupiter-5.4.2.jar!/" />
      <root url="jar://$MAVEN_REPOSITORY$/org/junit/jupiter/junit-jupiter-api/5.4.2/junit-jupiter-api-5.4.2.jar!/" />
      <root url="jar://$MAVEN_REPOSITORY$/org/apiguardian/apiguardian-api/1.0.0/apiguardian-api-1.0.0.jar!/" />
      <root url="jar://$MAVEN_REPOSITORY$/org/opentest4j/opentest4j/1.1.1/opentest4j-1.1.1.jar!/" />
      <root url="jar://$MAVEN_REPOSITORY$/org/junit/platform/junit-platform-commons/1.4.2/junit-platform-commons-1.4.2.jar!/" />
      <root url="jar://$MAVEN_REPOSITORY$/org/junit/jupiter/junit-jupiter-params/5.4.2/junit-jupiter-params-5.4.2.jar!/" />
      <root url="jar://$MAVEN_REPOSITORY$/org/junit/jupiter/junit-jupiter-engine/5.4.2/junit-jupiter-engine-5.4.2.jar!/" />
      <root url="jar://$MAVEN_REPOSITORY$/org/junit/platform/junit-platform-engine/1.4.2/junit-platform-engine-1.4.2.jar!/" />
    </CLASSES>
    <JAVADOC />
    <SOURCES />
  </library>
</orderEntry>
<orderEntry type="library" scope="TEST" name="Maven: org.junit.jupiter:junit-jupiter-api:5.7.1" level="project" />
<orderEntry type="library" scope="TEST" name="Maven: org.apiguardian:apiguardian-api:1.1.0" level="project" />
<orderEntry type="library" scope="TEST" name="Maven: org.opentest4j:opentest4j:1.2.0" level="project" />
<orderEntry type="library" scope="TEST" name="Maven: org.junit.platform:junit-platform-commons:1.7.1" level="project" />
<orderEntry type="library" scope="TEST" name="Maven: org.junit.jupiter:junit-jupiter-engine:5.7.1" level="project" />
<orderEntry type="library" scope="TEST" name="Maven: org.junit.platform:junit-platform-engine:1.7.1" level="project" />
```

모든 의존성을 삭제 후 다시 Junit5만 의존성 추가하여 테스트를 실행하니...

{% asset_img 성공.PNG org.junit.platform.commons.JUnitException: TestEngine with ID junit-vintage
  failed to discover tests %}

드디어 성공적인 테스트를 완료 할 수 있었다.

## 여담

project setting에서 분명 모듈 관련된 설정을 모두 삭제 했는데 왜 .iml파일에 설정이 삭제되지 않고 남아있는진 모르겠지만, 필자와 같은 상황이 발생하는 다른 사람들에게 도움이 되길 바라며 글을 마무리한다.