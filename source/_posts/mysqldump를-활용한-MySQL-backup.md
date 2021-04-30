---
layout: posts
title: mysqldump를 활용한 MySQL backup
date: 2021-04-30 23:02:05
tags:
    - db
    - mysql
    - 리눅스
---

## 서론

평소와 다름없이 유튜브를 배회하던 중 [재난급 서버 장애내고 개발자 인생 끝날뻔 한 썰](https://www.youtube.com/watch?v=SWZcrdmmLEU)이란 영상을 보게 되었는데... '나는 지금 어떻게 하고 있지..?' 라는 생각이 문득 들어서 얼른 나도 백업을 수시로 해둬야겠다는 생각에 적용한 mysql backup 방법을 남겨둔다.
* * *

## 왜 mysqldump로 했어요??

[mysqldump](https://dev.mysql.com/doc/refman/5.7/en/mysqldump.html)는 스토리지 엔진에 관계없이 논리 백업을 수행 할 수 있는 도구다.
무엇을 이용해서 backup을 해야할까 고민 하던 중 [장애와 관련된 XtraBackup 적용기](https://woowabros.github.io/experience/2018/05/28/billingjul.html)를 보게 되었는데
데이터 사이즈가 크지 않다면 mysqldump를 사용하는 것이 간단하며 복원 시에도 신경 써야 할 것이 적다는 내용을 보고 mysqldump로 backup을 진행하기로 결정했다.

## 그럼 어떻게 해요??

mysqldump엔 여러가지 옵션들이 있지만 그 중 필자는 아래 처럼 사용 했다.

```shell
mysqldump --single-transaction --databases [db명] --tables [테이블명] -h [db주소] -u [username] -p | gzip > [파일명].gz

```

### mysqldump option

- --single-transaction : lock 을 걸지 않고도 dump 파일의 정합성 보장하는데 InnoDB 테이블이 아닌 MyISAM or MEMORY 테이블인 경우에는 여전히 상태가 변경 될 수 있다.
MySQL에선 큰 테이블을 덤프하려면 --quick 옵션과 결합하기를 권장한다.
- --databases : dump 할 db명을 지정한다. 여러 개를 한번에 지정하는 것도 가능하다.
- --tables : dump 할 table명을 지정한다. 마찬가지로 여러 개를 한번에 지정 할 수 있다.

그리고 필자는 테이블 별로 데이터를 [gzip 형식](https://zetawiki.com/wiki/%EB%A6%AC%EB%88%85%EC%8A%A4_gzip)으로 압축하기 위해 gzip 명령어를 사용했다.

### 그 결과

{% asset_img 그_결과.PNG mysqldump를 활용한 MySQL backup 'mysqldump 실행 결과' 'mysqldump 실행 결과' %}

명령어를 실행하면 이 처럼 해당 경로에 .gz로 압축된 파일이 생성되고 이 파일의 압축을 해제하면 sql 파일이 된다.

### 예외 상황

mysqldump는 mysql  client 유틸리티 이므로 당연히 mysql client가 있어야 실행 가능한데 아래 메시지는 그 mysql client가 설치되지 않았으니 설치 한 후 사용하라는 메시지이다.

```shell
The program 'mysqldump' can be found in the following packages:
 * mysql-client-5.5
 * mariadb-client-5.5
 * mysql-client-5.6
 * percona-xtradb-cluster-client-5.5
Try: sudo apt-get install <selected package>
```

이 처럼 메시지가 출력 될 경우 위 리스트 중 하나를 설치하면 정상적으로 mysqldump가 이용 가능해 질 것이다.
* * *

## shell script로 작성하기

이제 이 명령어를 스케줄러에 등록하기 위해 우선 쉘 스크립트 파일로 작성해야 하는데 필자 같은 경우는 아래와 같이 작성해서 사용하고 있다.

```shell
#!/bin/bash
HOST=host명
USER=user명
PASSWORD=password
DATABASE=db명
TABLES=(
table1
table2
table3
...
)

TABLES_STRING=''
for TABLE in "${TABLES[@]}"
do :
   sudo mysqldump --single-transaction --databases ${DATABASE} --tables ${TABLE} --host ${HOST} --user ${USER} --password ${PASSWORD} | gzip > /home/backup/${TABLE}.sql.gz
done
```

이렇게 작성한 sh 파일을 bash 명령어로 실행하면 테이블 갯수만큼 비밀번호를 입력하면 백업 작업을 정상적으로 완료하게 된다... (정말?)
* * *

## crontab을 자동 backup

이제 백업을 자동으로 실행하게 하기 위해서 리눅스에 존재하는 [crontab]([https://zetawiki.com/wiki/리눅스_반복_예약작업_cron,_crond,_crontab](https://zetawiki.com/wiki/%EB%A6%AC%EB%88%85%EC%8A%A4_%EB%B0%98%EB%B3%B5_%EC%98%88%EC%95%BD%EC%9E%91%EC%97%85_cron,_crond,_crontab))을 이용하기로 한다.

### crontab 등록 방법

리눅스 서버에 접속해서

```shell
crontab -l
```

명령어를 실행하면 아래 처럼 **해당 유저**로 서버에 등록된 스케줄들이 보이게 되는데 해당 유저로 등록된 서비스가 없다면 아래 처럼

```shell
user명@ip-127.0.0.1:~$ crontab -l
no crontab for user명
```

메시지가 출력되고 해당 유저로 등록된 서버스가 존재 할 경우엔

```shell
user명@ip-127.0.0.1:~$ crontab -l
# Edit this file to introduce tasks to be run by cron.
# 
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
# 
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').# 
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
# 
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirected).
# 
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
# 
# For more information see the manual pages of crontab(5) and cron(8)
# 
# m h  dom mon dow   command
10 04 * * * bash 백업 실행 파일.sh
```

위 처럼 cat 명령어를 실행 한 것 처럼 콘솔에 crontab의 내용이 보여지게 된다.
이렇게 저장된 스케줄러들을 확인난 후에

```shell
crontab -e
```

명령어를 통해 스케줄러를 등록하면 되는데 이 때 crontab -e 명령어를 실행시키면 crontab을 열 텍스트 에디터를 선택하라는 메시지가 출력되거나, 디폴트로 지정된 텍스트 에디터로 crontab이 열리게 된다.

어떤 식이든 crontab이 열리면

```shell
* * * * *  수행할 명령어
┬ ┬ ┬ ┬ ┬
│ │ │ │ │
│ │ │ │ │
│ │ │ │ └───────── 요일 (0 - 6) (0:일요일, 1:월요일, 2:화요일, …, 6:토요일)
│ │ │ └───────── 월 (1 - 12)
│ │ └───────── 일 (1 - 31)
│ └───────── 시 (0 - 23)
└───────── 분 (0 - 59)
```

위 형식에 맞춰 언제마다 실행 시킬 지 정한 후 우리가 만든 backup용 sh 파일을 실행하게 하면 된다.

필자는 매일 04시 10분에 실행 시키기 위해 아래와 같이 작성하여 등록했다.

```bash
10 04 * * * bash 백업 실행 파일.sh
```

이렇게 작성하고 테스트를 하기 위해 date 명령어로 서버 시간을 확인한 후 실행 시간을 현재 시간에서 2분 뒤로 변경하여 2분 뒤에 실행하게 하였는데...
* * *

## 왜 백업이 안되지...?

{% asset_img 백업실패.PNG mysqldump를 활용한 MySQL backup '뎃? 어쨰서 파일 사이즈가...?' '뎃? 어쨰서 파일 사이즈가...?' %}

