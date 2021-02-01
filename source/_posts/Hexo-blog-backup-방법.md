---
layout: posts
title: Hexo blog backup 방법
date: 2021-01-02 15:54:05
tags:
---

## 서론
많은 블로그 서비스 중 필자는 Git, Github를 활용해서 사용하는 hexo로 github 블로그를 선택했지만, 다른 블로그 서비스들과는 달리 설치형 블로그의 특성상 설치된 PC가 아닌 경우엔 블로그에 글을 작성하는 일이 쉽지 않고, 블로그가 설치된 PC가 고장이라도 나는 날에는 고스라니 블로그를 날려버리는 상황이 닥치는 경우도 왕왕있다. (예로 필자는 블로그를 설치한 HDD가 고장나 블로그를 새로 작성했다.) 이런 경우를 방지하고, 블로그 작성의 확장성을 위해 github을 이용해 Hexo 블로그를 back up 하는 방법을 알아본다.
* * *

## 방법

{% asset_img 1_저장소.PNG Hexo blog backup 방법 "post와 테마를 저장 할 저장소 생성" "'post와 테마를 저장 할 저장소 생성'" %}

1. 우선 위 사진처럼 github에서 테마와 post를 저장 할 repository를 생성한다.
이 때 테마를 저장 할 저장소의 이름은 테마와 같게 한다.

{% asset_img 2_테마_저장소_변경.PNG Hexo blog backup 방법 "설치한 테마의 저장소 변경" "'설치한 테마의 저장소 변경'" %}

2. 테마를 저장하기 위해 연결된 원격 저장소의 위치를 확인하고 설치한 테마 폴더 안에서 git 원격 저장소 주소를 변경한다.
3. 변경 후 테마용 저장소에 테마를 push한다.

{% asset_img 3_테마-저장확인.PNG Hexo blog backup 방법 "push 된 테마 확인" "'push 된 테마 확인'" %}

4. 위 처럼 정상적으로 저장되어 있는지 확인하고 themes 폴더안에서 테마를 삭제한다.
5. git의 submodule 기능을 이용해 저장한 테마를 submodule로 추가하는데 이 때 디렉토리의 위치는 themes 폴더 내로 이동해서 submodule 기능을 실행한다.

```git
git submodule add 테마 저장소
```

6. post를 저장 할 저장소 주소를 추가한다.(cotent)

```git
git remote add content post 저장소
```

{% asset_img 4_post-저장확인.PNG Hexo blog backup 방법 "post 저장 확인" "'post 저장 확인'" %}

7. post 저장용 저장소에 내용을 push하고 정상적으로 저장되었는지 확인한다.
8. 이 후엔 post 작성 후 post만 content 저장소로 push해주면 된다.
* * *

## 결과

이제 우리는 불미스러운 사고로 블로그가 설치된 HDD가 고장나거나(ㅠㅠ), 블로그가 설치 안된 다른 PC or 노트북에서도 백업된 theme와 post들을 다운받아 블로그를 작성 할 수 있게 되었다.

여담이지만, 과거로 돌아간다면 나는 velog를 사용 할 듯 싶다.
* * *

## 참고 사이트

* [Hexo 배포 원리와 백업하기](https://futurecreator.github.io/2018/07/18/hexo-blog-backup/)
* [Git cached 삭제](https://blog.naver.com/PostView.nhn?blogId=bestmic&logNo=220939712681&proxyReferer=https:%2F%2Fwww.google.com%2F)
* [Git Submodule 삭제방법](http://snowdeer.github.io/git/2018/08/01/how-to-remove-git-submodule/)
* [Git: fatal: Pathspec is in submodule](https://stackoverflow.com/questions/24472596/git-fatal-pathspec-is-in-submodule)
