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

{% asset_img 그_결과.png mysqldump를 활용한 MySQL backup 'mysqldump 실행 결과' 'mysqldump 실행 결과' %}

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

{% asset_img 백업실패.png mysqldump를 활용한 MySQL backup '뎃? 어쨰서 파일 사이즈가...?' '뎃? 어쨰서 파일 사이즈가...?' %}

이미지에서 보이는 것 처럼 파일 사이즈가 20 바이트 밖에 안되길래 확인해보니 백업이 정상적으로 이루어지지 않은 것 이었다.

왜 이렇게 됐을까를 고민하던 중... 스케줄러에 등록하기 전 sh파일을 단독으로 실행 했을 때 table 갯수만큼 비밀번호를 입력했던 것이 떠올라서 그 부분이 문제일 것이라고 판단되어 찾아보니 [공식 문서]([https://dev.mysql.com/doc/refman/8.0/en/password-security-user.html](https://dev.mysql.com/doc/refman/8.0/en/password-security-user.html))에서 아래와 같은 글을 확인 할 수 있었다.

{% blockquote Seth Godin https://dev.mysql.com/doc/refman/8.0/en/password-security-user.html 6.1.2.1 End-User Guidelines for Password Security %}
Use the --password or -p option on the command line with no password value specified. In this case, the client program solicits the password interactively:

shell> mysql -u francis -p db_name
Enter password: ********
The * characters indicate where you enter your password. The password is not displayed as you enter it.

It is more secure to enter your password this way than to specify it on the command line because it is not visible to other users. However, this method of entering a password is suitable only for programs that you run interactively. If you want to invoke a client from a script that runs noninteractively, there is no opportunity to enter the password from the keyboard. On some systems, you may even find that the first line of your script is read and interpreted (incorrectly) as your password.
{% endblockquote %}

정리하면 —password 옵션은 클라이언트에 대화식으로 암호를 요청하고, 이 경우 비대화형으로 실행되는 스크립트에서 클라이언트를 호출 할 경우 암호를 입력 할 기회가 없으므로 일부 시스템에서 잘못 해석 될 수도 있다는 내용이었다.

### 그래서 어떻게 해야되요...

공식 문서를 보면 .my.cnf파일을 만들어서 거기에

```shell
[client]
password=password
```

아래와 같은 형식으로 지정하고 안전을 위해 파일의 엑서스 모드를 400 또는, 600으로 설정하라고 한다.

그 후 mysqldump에서 지정한 설정 파일을 읽어들이게 하기 위해

```shell
shell> mysql --defaults-file=/home/francis/mysql-opts
```

처럼 파일의 전체 경로를 지정하라고 설명하고 있다.

### 바뀐 sh파일 내용

```bash
#!/bin/bash
HOST=host명
USER=user명
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
   sudo mysqldump --single-transaction --databases ${DATABASE} --tables ${TABLE} --host ${HOST} --user ${USER} | gzip > /home/backup/${TABLE}.sql.gz
done
```

—password 옵션을 제외하고 password가 my.cnf 설정파일에 들어갔으므로 최종적인 sh 파일 내용은 위와 같이 된다.

## 결과

{% asset_img 성공.png mysqldump를 활용한 MySQL backup '성공, 대성공!' '성공, 대성공!' %}

모든 난관을 극복하고 다시 crontab -e에 등록해서 스케줄을 실행한 결과 위와 같이 정상적으로 백업을 완료하여 파일 사이즈가 아까와 다르게 20 byte가 아닌 모습을 볼 수 있다.
* * *

## P.S : 저는 그래도 안되는데요...?

필자의 경우에도 .my.cnf 파일을 생성하고 등록해줬음에도 불구하고 백업이 정상적으로 진행되지 않았는데 결국 찾은 해결 방법은 .my.cnf 파일을 따로 생성하지 않고, 아래와 같이 /etc/mysql/my.cnf 파일에 password를 추가하는 방법으로 해결 했다.

```shell
...

# One can use all long options that the program supports.
# Run program with --help to get a list of available options and with
# --print-defaults to see which it would actually understand and use.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

# This will be passed to all mysql clients
# It has been reported that passwords should be enclosed with ticks/quotes
# escpecially if they contain "#" chars...
# Remember to edit /etc/mysql/debian.cnf when changing the socket location.
[client]
port		= 3306
socket		= /var/run/mysqld/mysqld.sock
password        = password # !!! 이 부분 !!!
# Here is entries for some specific programs
# The following values assume you have at least 32M ram

# This was formally known as [safe_mysqld]. Both versions are currently parsed.
[mysqld_safe]
socket		= /var/run/mysqld/mysqld.sock
nice		= 0

...
```

필자와 같은 경우도 아니라면... 행운을 빈다...
* * *

## 참고 사이트

- [MySQL 공식 사이트 mysqldump document](https://dev.mysql.com/doc/refman/5.7/en/mysqldump.html)
- [우아한 형제들 기술 블로그 - 장애와 관련된 XtraBackup 적용기](https://woowabros.github.io/experience/2018/05/28/billingjul.html)
- [mysqldump 사용법(db backup 및 load 하기)](https://www.lesstif.com/dbms/mysqldump-db-backup-load-17105804.html)
- [MySQL 데이터 dump(Export)/Import 하기](https://musma.github.io/2019/02/12/mysql-dump-and-import.html)
- [How to dump database and ignore some tables with mysqldump?]([https://tableplus.com/blog/2018/08/mysql-how-to-dump-database-ignore-tables-or-data-with-mysqldump.html](https://tableplus.com/blog/2018/08/mysql-how-to-dump-database-ignore-tables-or-data-with-mysqldump.html))
- [End-User Guidelines for Password Security]([https://dev.mysql.com/doc/refman/8.0/en/password-security-user.html](https://dev.mysql.com/doc/refman/8.0/en/password-security-user.html))
