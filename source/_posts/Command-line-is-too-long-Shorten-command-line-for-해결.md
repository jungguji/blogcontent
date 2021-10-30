---
layout: posts
title: Command line is too long. Shorten command line for.. 해결
date: 2021-05-30 11:48:52
tags:
    - intellij
    - junit
---

## 상황

평소와 같이 테스트 코드를 작성하고 실행을 했는데... 아래와 같이 에러가 발생하며 테스트가 진행되지 않았다.

### 발생한 에러

{% asset_img 에러.JPG Command line is too long. Shorten command line for.. 해결 %}

* * *

## 해결

{% asset_img workspace.JPG Command line is too long. Shorten command line for.. 해결 %}

위 이미지처럼 프로젝트 루트에 존재하는 .idea 폴더에 workspace.xml 파일을 열어서

```xml
<component name="PropertiesComponent">
    <property name="dynamic.classpath" value="true" />
</component>
```

를 추가해주면 된다.

보통 이미 PropertiesComponent 태그는 존재 할 테니 그 아래 있는

```xml
<property name="dynamic.classpath" value="true" />
```

옵션만 정상적으로 넣어주면 된다.

* * *

## 보너스

### dynamic.classpath 옵션이 뭔데 넣으면 되는 거에요?

이 질문에 대한 답은 역시 없는게 없는 스택오버 플로우에 이미 존재한다. [What does the dynamic.classpath flag do? (IntelliJ project settings)](https://stackoverflow.com/questions/4853540/what-does-the-dynamic-classpath-flag-do-intellij-project-settings)

간단하게 정리하면 커맨드라인이나 파일을 통해 클래스 패스가 jvm에 전달되는 방식을 제어하는데 대부분의 os에선 최대 명령 줄 제한이 있어서 이 제한을 넘는 명령은 IDEA에서 실행 할 수 없기 때문에 명령이 32768 자 보다 길면 동적 클래스 패스로 전환 할 것을 제안하고 이 때 긴~ 클래스 패스는 파일에 기록된 다음, 응용 프로그램 관리자가 읽어서 시스템 클래스 로더를 통해 로드 된다는 것이다.

끝!

* * *

## 참고 사이트

- [https://stackoverflow.com/questions/4853540/what-does-the-dynamic-classpath-flag-do-intellij-project-settings](https://stackoverflow.com/questions/4853540/what-does-the-dynamic-classpath-flag-do-intellij-project-settings)
- [http://jmlim.github.io/intellij/2020/02/27/intellij-idea-command-line-is-too-long-error/](http://jmlim.github.io/intellij/2020/02/27/intellij-idea-command-line-is-too-long-error/)