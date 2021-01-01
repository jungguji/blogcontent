---
layout: posts
title: Spring Boot + Maven Travis CI 연동 에러
date: 2020-06-06 20:17:56
tags:
    - Spring-Boot
    - Travis CI
---

## 에러

![Travis 권한 에러](/images/20200601/travis_error.PNG)

```text
./mvnw: Permission denied
```

Spring Boot + Maven 프로젝트 Travis CI 연동 중 발생한 에러

* * *

## 해결

```text
before_install:
  - chmod +x mvnw
```

설정으로 권한 부여하여 해결

## 전체 코드

```text
language: java
jdk:
  - openjdk8

branches:
  only:
    - master

# Travis CI 서버의 Home
cache:
  directories:
    - $HOME/.m2

before_install:
  - chmod +x mvnw

# MAVEN 프로젝트는 필요 없음
#script: "./mvnw package"

# CI 실행 완료시 메일로 알람
notifications:
  email:
    recipients:
      - your@gmail.com
```

* * *

## 참고 사이트

- [https://velog.io/@recordsbeat/travis-ci-maven-%EC%97%B0%EB%8F%99](https://velog.io/@recordsbeat/travis-ci-maven-%EC%97%B0%EB%8F%99)
