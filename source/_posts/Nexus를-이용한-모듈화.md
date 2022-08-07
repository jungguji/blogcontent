---
layout: posts
title: Nexus를 이용한 모듈화
date: 2022-08-07 20:14:31
tags:
    - nexus
    - spring
    - 모듈화
---

## 서론

 현재 운영 중인 서비스는 같은 데이터베이스를 공유하는 5개의 프로젝트가 존재하는데, 모두 Spring or Spring boot + JPA를 사용하는 프로젝트들이다.
 이 때문에, 한 프로젝트에서 도메인(Entity) 클래스에 변수를 추가한 경우에 다른 모든 프로젝트에서도 동일하게 추가 해주지 않으면 데이터베이스 조회 시 등 에러가 발생 할 수 밖에 없는데 이런 copy & paste 반복 작업을 개발자가 손수 하다보면 휴먼에러가 발생할 수 밖에 없음은 물론 같은 데이터베이스를 사용하는 Entity 임에도 불구하고 코드가 모두 다른 이상한 상황이 발생 할 수 밖에 없는 구조였다.
 그렇기에 어떻게 할까 고민하던 중 [멀티모듈 설계 이야기 with Spring, Gradle](https://techblog.woowahan.com/2637/) 이라는 글을 보고 우리 서비스에도 단일 프로젝트 멀티모듈 구조로 가져갈 까 하다, 각각의 프로젝트마자 깃을 따로 관리하는게 우리 서비스에는 맞다는 생각이 들어 멀티 프로젝트에 Nexus maven 저장소를 이용해서 도메인 클래스들을 분리 시키기로 했다.

* * *

## 본론

### 현재 시스템의 구조

{% asset_img "현재_구조.png" "현재 서비스 구조" %}

이 처럼 여러 개의 프로젝트가 하나의 데이터베이스를 공유하는 구조로, 같은 데이터베이스 이므로 동일한 내용의 도메인 클래스가 각자 프로젝트에 각각 중복으로 존재하는 구조이기에, 도메인 클래스들을 새로운 프로젝트에 몰아넣고 Jar 파일로 라이브러리화해서 내부 maven 저장소인 [Nexus](https://www.sonatype.com/products/nexus-repository)에 업로드하고, 다른 프로젝트들에서 라이브러리화해서 업로드된 도메인 모듈을 의존성에 추가하여 시스템적으로 정합성을 유지하면서 사용 할 수 있게 한다.

간단하게 정리하면

1. Nexus에 모듈화된 도메인 프로젝트 라이브러리 업로드
2. 다른 프로젝트들에서 도메인 라이브러리 디펜던시 추가하여 사용

하는 것이 이번 글의 목표이다.

- 이 글엔 Nexus 내부 maven 저장소를 설치하는 내용은 담지 않았으므로, 다른 분들이 작성하신 글을 보고 자신의 환경에 맞게 설치 해야한다.

### 1. Nexus 설정

다른 분들의 글을 보고 무사히 설치를 완료하고 실행하면 아래와 같은 화면을 볼 수 있을 것이다.

{% asset_img "nexus_첫화면.png" "설치 후 첫 화면" %}

그 후 Sign in 버튼을 클릭하면 초기 아이디와 비밀번호가 저장된 파일의 위치를 알려준다.

{% asset_img "로그인화면.png" "로그인 화면" %}

```shell
cat /설치경로/nexus3/admin.password
```

{% asset_img "초기_패스워드.png" %}

설정 하다보면 아래처럼 익명 사용자도 접근하게 할 것인지 물어보는데 당연히도 수정 할 수 있으므로 적당히 선택해주면 된다.

{% asset_img "익명유저_접근.png" %}

### 2. Repository 생성

우선 모듈화한 도메인 클래스들을 업로드 할 레파지토리부터 생성한다.

{% asset_img "레파지토리_생성1.png" %}

{% asset_img "레파지토리_생성2.png" %}

레파지토리 type은 group, hosted, proxy가 있지만, 우리는 위 처럼 maven2의 hosted로 생성한다.

- 이때 hosted는 레파지토리 관리자가 직접 호스팅하는 레파지토리 type이고,
- 다른 것들과의 차이점은 [여기서](https://help.sonatype.com/repomanager2/configuration/managing-repositories) 확인 해 볼 수 있다.

{% asset_img "레파지토리_생성3.png" %}

{% asset_img "레파지토리_생성_version_policy.png" %}

- Version policy는
  - Release
  - Snapshot
  - Mixed

 3 가지가 존재하는데, 보통 Snapshot 레파지토리, Release 레파지토리를 따로 만들어 관리해야하지만, 우리는 Mixed로 지정하고 사용하였다.

{% asset_img "레파지토리_생성_deployment_policy.png" %}

- Deployment policy는
  - Allow redeploy
    - 같은 버전으로 재배포 가능하고 덮어씀
  - Disable redeploy
    - 같은 버전으로 다시 배포 할 수 없음
  - Read-only
    - 아예 배포가 허용되지 않음
  - Deploy by Replication Only
    - 레플리카를 사용하는 경우 레플리카로만 배포되고 다른 배포는 차단됨

- 자세한 내용은 [여기서](https://help.sonatype.com/repomanager3/nexus-repository-administration/repository-management#RepositoryManagement-Hosted) 확인 할 수 있다.

### 3. Repository access 유저 생성

### Role 생성

{% asset_img "Role생성1.png" %}

{% asset_img "Role생성2.png" %}

필터란에 생성한 레파지토리명으로 검색해서 - nx-repository-view-… 로 시작하는 권한들을 넣어주고, 해당 Role로 레파지토리에 업로드도 할 수 있게 하기 위해 nx-component-upload 권한도 넣어준다.

### 유저 생성

{% asset_img "유저생성1.png" %}

{% asset_img "유저생성2.png" %}

각 항목들은 각자에 맞게 넣어주고 Role엔 조금 전에 생성한 Role을 넣어주고 생성을 완료하면 우리가 처음에 로그인한 어드민 계정과는 달리 우리가 생성한 repository만 접근 할 수 있는 것을 확인 할 수 있다.

{% asset_img "유저생성3.png" %}

* * *

### 4. 라이브러리 업로드

### settings.xml 설정

nexus 레파지토리에 접근하기 위해서는 우선, maven 설정 파일인 settings.xml을 수정해줘야 한다. 기본적으로는 ~/.m2/settings.xml 으로 설정되어 있으므로 이 곳에 생성해서 사용하거나, 이미 따로 지정한 곳에 위치한 settings.xml을 사용하고 있다면 그 위치의 settings.xml 파일을 수정해주면 되는데.. 이도 저도 모르겠으면 아래 이미지 처럼 인텔리제이에서 확인하여 그 위치의 settings.xml을 수정해준다.

{% asset_img "settings.png" %}

