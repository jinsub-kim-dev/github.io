---
layout: post
title: "라즈베리파이에 개발 DB 구축하기"
date: 2020-08-10 23:35:00 +0900
categories: Database
tags: MySQL Database Raspberry-Pi
---

## 외부 데이터베이스 설정의 필요성

데이터베이스 서버를 외부에 따로 설정하지 않으면 로컬 데이터베이스를 통해 프로젝트를 진행하게 된다. 로컬 데이터베이스는 직접 접근이 가능하다는 장점이 있다. 그러나 외부 개발 환경과 데이터베이스를 공유할 수 없으며, 다른 개발 환경에서 작업하려면 데이터베이스를 동기화 시켜야 한다. 또한 본인처럼 로컬 컴퓨터를 포맷할 경우 (백업이 되어있지 않는다면) 데이터베이스 테이블과 데이터를 처음부터 다시 구축해야 한다. 여기선 외부에 데이터베이스 서버를 구축하는 방법을 다룰것이며 서버 디바이스로 `라즈베리파이 3 모델 B`를 사용한다.

## 라즈베리파이에 MySQL 설치하기

라즈비안 OS 설치와 각종 라즈베리 파이 설정(SSH, 한글, 고정 IP 등 필수는 아닌 설정들)이 되어 있다고 가정한 후 MySQL 설치 과정을 진행한다.

```bash
# 패키지 업데이트
$ sudo apt-get update
$ sudo apt-get upgrade

# MySQL 설치
$ sudo apt-get install mysql-server mysql-client

# root 계정으로 MySQL 접속
$ sudo mysql -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 13
Server version: 10.1.45-MariaDB-0+deb9u1 Raspbian 9.11

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
```

위와 같이 root 계정으로 접속할 수 있다면 설치는 완료되었다.

## 라즈베리파이 데이터베이스의 외부 접근 허용 (1) - 설정 변경

라즈베리파이에 데이터베이스를 설치한 이유는 로컬 개발 환경과 분리된 서버를 갖추기 위함이었다. 따라서 외부에서 데이터베이스에 접근할 수 있도록 설정을 변경해야 한다.
`/etc/mysql/mariadb.conf.d/` 경로에 여러 설정 파일이 존재한다. 그 중 `50-server.cnf` 파일을 열어 `bind-address = 127.0.0.1` 부분을 **주석 처리** 해준다.

```bash
$ sudo cat /etc/mysql/mariadb.conf.d/50-server.cnf
... 생략 ...
[mysqld]

#
# * Basic Settings
#
user            = mysql
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
port            = 3306
basedir         = /usr
datadir         = /var/lib/mysql
tmpdir          = /tmp
lc-messages-dir = /usr/share/mysql
skip-external-locking

# Instead of skip-networking the default is now to listen only on
# localhost which is more compatible and is not less secure.
# bind-address          = 127.0.0.1
... 생략 ...
```

설정 파일을 수정한 후 MySQL을 재시작한다.

```bash
$ sudo service mysql restart
```

## 라즈베리파이 데이터베이스의 외부 접근 허용 (2) - 외부 접근 계정 생성

MySQL 계정 정보를 확인해보면 `root`라는 이름의 계정으로 로컬에서만 데이터베이스에 접근할 수 있다는 사실을 알 수 있다.

```bash
$ sudo mysql -u root -p --silent
Enter password:
MariaDB [(none)]> USE mysql;
MariaDB [mysql]> SELECT host,user FROM user;
+-----------+-------+
| host      | user  |
+-----------+-------+
| localhost | root  |
+-----------+-------+
```

여기선 `housekeeping` 이라는 이름의 데이터베이스를 추가하고, 개발에 사용될 계정 생성 및 권한을 부여한다. 또한 새로 추가한 계정으로 외부 접근이 가능하도록 설정을 변경한다.

```bash
# 데이터베이스 생성
MariaDB [mysql]> CREATE DATABASE housekeeping;

# 'swiri'라는 이름의 유저 생성. 비밀번호는 'abc1234'로 지정한다.
MariaDB [mysql]> CREATE USER 'swiri'@'%' IDENTIFIED BY 'abc1234';
MariaDB [mysql]> SELECT host,user FROM user;
+-----------+-------+
| host      | user  |
+-----------+-------+
| %         | swiri |
| localhost | root  |
+-----------+-------+

# 'swiri' 유저에게 'housekeeping' 데이터베이스에 대한 모든 권한을 부여한다.
MariaDB [mysql]> GRANT ALL ON housekeeping.* TO 'swiri' IDENTIFIED BY 'abc1234';
MariaDB [mysql]> FLUSH PRIVILEGES;

# MySQL 재시작
$ sudo service mysql restart
```

## 외부에서 원격 데이터베이스에 접근

원격 데이터베이스에 접근하기 위한 클라이언트 프로그램으로 `MySql Workbench`를 사용한다. 상단 메뉴에서 `Database > Connect to Database`를 통해 다음과 같이 원격 접속을 진행한다.
이후 해당 계정을 통해 원격 데이터베이스에 접근할 수 있다.

![image](/post_assets/2020-08-10/database_connection.png)
