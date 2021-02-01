---
layout: posts
title: Spring boot Heroku에 배포하기
date: 2021-01-28 21:07:20
tags:
    - Spring boot
    - Heroku
    - Could
---

## 서론

AWS 프리티어는 이미 다른 서비스가 이용하고 있고 Java를 지원해주는 클라우드 서비스가 없었는데 [heroku](https://www.heroku.com/)를 발견하여 사용방법을 정리 해둔다.

__참고로 데이터베이스는 한글을 지원하지 않는다.__
* * *

## 본론

1. heroku 회원 가입

{% asset_img 회원가입.PNG Spring boot Heroku에 배포하기 "회원가입" "회원가입" %}

2. App 생성

{% asset_img new_app.PNG Spring boot Heroku에 배포하기 "Create a new app" "Create a new app" %}

위에 보이는 create a new app 버튼을 클릭하고

{% asset_img image.png Spring boot Heroku에 배포하기 "app 이름 지정" "app 이름 지정" %}

App name을 지정해야 하는데 이미 heroku에 존재하는 service의 name은 생성 할 수 없고,
여기서 지정한 App name으로 호스팅 될 URL이 생성된다.

- ex) appname : backjoonframeautomaticgenerat
    URL : [https://backjoonframeautomaticgenerat.herokuapp.com/](https://backjoonframeautomaticgenerat.herokuapp.com/)

3. heroku CLI 설치

{% asset_img CLI_설치.png Spring boot Heroku에 배포하기 "Heroku CLI 설치" "Heroku CLI 설치" %}

생성이 완료되면 위 화면처럼 Deploy tab으로 이동되는데 설명되어 있는 것 처럼 먼저 [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli)를 자신의 OS 버전에 맞게 설치한다.

4. Procfile 생성

Heroku는 실행 할 때 마다 port를 자동으로 지정해주는데 port를 고정시키기 위해 우선 [application.properties에 port를 바인딩](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto-use-short-command-line-arguments) 해준다.

### application.properties

```java
server.port=${port:8080}
```

그 후 Procfile을 Project 루트 디렉토리에 확장자 없이 생성하고 아래와 같이 작성한다.

```java
web: java -Dserver.port=$PORT $JAVA_OPTS -jar [실행될 jar파일 경로]
```

### Procfile의 경로

{% asset_img Procfile_경로.png Spring boot Heroku에 배포하기 "Procfile의 경로" "Procfile의 경로 루트 디렉토리에 오도록 작성한다." %}

5. 배포

{% asset_img 배포.png Spring boot Heroku에 배포하기 "배포" "배포" %}

이후엔 heroku의 가이드를 그대로 따라하면 된다.

{% asset_img 배포완료.png Spring boot Heroku에 배포하기 "배포 완료" "배포 완료" %}

모든 가이드를 정상적으로 잘 따라하면 위 처럼 접속할 수 있는 URL이 출력되고 해당 URL로 접속하면

6. 확인

[https://backjoonframeautomaticgenerat.herokuapp.com/](https://backjoonframeautomaticgenerat.herokuapp.com/)

{% asset_img 확인.PNG Spring boot Heroku에 배포하기 "확인" "배포가 마무리된 모습" %}

정상적으로 실행되어 서비스가 실행되는 것을 확인 할 수 있다.
* * *

## 결론

데이터베이스를 사용하지 않는 서비스나, 한글이 입력되지 않는 서비스의 경우 무료로 이용 할 수 있는 좋은 클라우드 서비스 인 것 같다.

참고로 30분간 접속이 없으면 휴면 모드로 전환되어 최초 접속이 다소 느릴 수 있으나 무료 서비스인 만큼 그정도는 감안해주자.

이 글에 소개한 CLI를 이용한 방법 말고 Github와 연결해서 branch에 push하면 자동으로 배포되게 할 수 도 있는 것 같으니 찾아보고 적용하면 이 방법 보다 더 편할 것이다.
* * *

## 참고 사이트

- [https://jeong-pro.tistory.com/182](https://jeong-pro.tistory.com/182)
- [https://wedul.site/590](https://wedul.site/590)