---
title: 파워쉘로 MYSQL 접속하기
date: 2020-07-25-14:00
categories:
- DB

tags:
- power shell
- MYSQL
- DB

photos: 
- /post_images/mysql.png

---

## 파워쉘을 이용해 MYSQL에 접속해봅니다.
> `bitnami`를 설치하며 같이 설치된 mysql을 이용해봅니다.

---

## mysql.exe가 있는 경로로 이동하고 mysql 명령을 실행합니다.
> :heavy_check_mark: `./mysql.exe -uroot -p`

![접속](/post_images/mysql_0.jpg)

* `-` 으로 옵션을 지정할 수 있습니다.
* `-uroot`는 데이터 베이스 사용자를 뜻합니다.
* `-p`는 경우 패스워드를 뜻합니다.

## 비밀번호를 입력합니다.

![접속](/post_images/mysql_1.jpg)

* 비밀번호를 이어 입력하면 접속이 됩니다.

---

## 데이터 베이스 목록을 봅니다.
> :heavy_check_mark: `show databases;`

![접속](/post_images/mysql_2.jpg)

* 아직 이상태는 `DB`에 접속한 상태가 아닙니다.
* `MYSQL` 서버에만 접속한 상황입니다.

## 데이터 베이스를 선택합니다.
> :heavy_check_mark: `use sample`

![접속](/post_images/mysql_3.jpg)

* 이떄 `use`는 `mysql` 클라이언트의 명령이므로, `SQL` 명령이 아니라 `;`를 생략해도 됩니다.
    * `;`를 포함하여도 문제는 발생하지 않습니다.

---

## MYSQL 접속과 동시에 DB도 접속할 수 있습니다.
> :heavy_check_mark: `./mysql.exe -uroot -p비밀번호 DB이름`

![한번에 접속](/post_images/mysql_4.jpg)

* `-p비밀번호` 에서 띄어쓰기를 하지 않도록 주의합니다.
* 명령이 기록에 남을 경우, 보안상 `비밀번호가 노출` 될 수 있으므로 경고 메시지가 나옵니다.

---

## Reference
* [SQL 첫걸음](http://www.yes24.com/Product/Goods/22744867)