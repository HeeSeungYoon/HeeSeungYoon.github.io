---
layout: post
title: JetSon Nano OS(Linux)
date: 2023-11-20 00:00:00 +0900
description: JetSon Nano에서 Linux Setting
img: # Add image post (optional)
tags: [JetSon Nano, Linux, Setting] # add tag
---

# Python Package 관리

## Python version 변경

JetSon Nano의 default python 버전은 python 2.7과 python 3.6 이다.

Python의 다른 버전이 필요한 경우 

1. apt install python3.8
2. sudo update-alternatives --install "link" "name" "path" "priority"
	- link: 실행할 파일 이름 e.g. /usr/bin/python
	- name: 해당 링크 그룹의 대표 이름 
	- path: 실제 연결할 실행 파일 이름 e.g. /usr/bin/python3.8
	- priority: auto 모드에서 어떤 패키지를 자동으로 선택할지 결정할 때 사용하는 우선순위, 높은 수가 높은 우선 순위
3. sudo update-alternatives --config "name"으로 패키지 리스트 확인 가능
	
    ![update alternative]({{site.baseurl}}/assets/img/linux/update_alternatives.png)

---

# Samba

Windows 운영체제를 사용하는 PC에서 Linux 또는 UNIX 서버에 접속하여 파일이나 프린터를 공유하여 사용할 수 있도록 해 주는 소프트웨어
## 1. apt install samba

## 2. window와 공유할 directory 생성

- e.g. mkdir /home/"id"/Desktop/"samba directory"

## 3. samba share 설정

- /etc/samba/smb.conf 파일의 마지막에 아래의 코드 추가
	
    ![smb.conf]({{site.baseurl}}/assets/img/linux/sambashare.png)
    - path: 2에서 생성한 directory path
    - valid users: 본인 id
    - create mask, directory mask: 사용 권한 (-rwxrwxrwx)

## 4. window 네트워크 드라이브 연결

1. window 파일 탐색기에서 네트워크 탭 우클릭
2. 네트워크 드라이브 연결 클릭
3. \\\\"your ip address"\\sambaShare 입력

---

# Maria DB

## 1. apt install mariadb-server

## 2. sudo mysql

- maria DB 접속
	- DB 유저 추가: create user 'user id'@'%' identified by 'passwd';
	- 유저 권한 부여: grant all privileges on \*.\* to 'user id'@'%'; 

## 3. maria DB conf 파일 수정 

- 다른 ip에서 maria DB로 접속 가능하도록 conf 파일 수정
	- /etc/mysql/mariadb.conf.d/50-server.cnf 파일에 bind-address가 localhost(127.0.0.1)로 되어있다면 주석 처리
- mysql 서버 재시작
	- sudo service mysql restart 

## 4. Window에서 [MySQL Workbench](https://dev.mysql.com/downloads/file/?id=519997) 설치

## 5. MySQL Connection

- Connection Name, Hostname, Username 입력 후 Test Connection 실행
    
    ![mysql workbench]( {{site.baseurl}}/assets/img/linux/mysql_workbench.png)

