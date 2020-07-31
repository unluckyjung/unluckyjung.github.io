---
title: ubuntu ssh 포트 열고 접속하기
date: 2020-07-31-16:00
categories:
- Linux

tags:
- linux
- ubuntu

photos: 
- https://user-images.githubusercontent.com/43930419/88630201-c57e7d00-d09f-11ea-85ea-a479d8e0a3eb.png

---

## 우분투에서 ssh 포트를 열고, 외부에서 접속해 봅니다.
> `putty`를 이용해 접속해 봅니다.

---

## 설치 되어있는 패키지를 업데이트 합니다.

![이미지](https://user-images.githubusercontent.com/43930419/88630173-bf889c00-d09f-11ea-8a77-43d91624c758.png)

```shell
sudo apt update
```

* `update`와 `upgrade` 차이 [링크](https://twpower.github.io/65-difference-between-update-and-upgrade)

---

## SSH 서버를 설치 하고 포트를 연뒤 방화벽까지 해제해 줍니다.

![이미지](https://user-images.githubusercontent.com/43930419/88630176-c0b9c900-d09f-11ea-8eee-9ce08f008086.png)

```shell
sudo apt install openssh-server
```

![이미지](https://user-images.githubusercontent.com/43930419/88630177-c0b9c900-d09f-11ea-8b93-e277e30e085c.png)

```shell
sudo systemctl status ssh
```

![이미지](https://user-images.githubusercontent.com/43930419/88630178-c1525f80-d09f-11ea-92c3-4036d1e5c408.png)

```shell
sudo ufw allow ssh
```

### SSH 란?
> `Secure Shell Protocol`

* 컴퓨터간 연결을 `public network`로 할때, 안전하게 통신하기 위한 `protocol`
* `공개키` `비공개키` 를 전부 이용하여 통신한다.
* [과거 내가 공부한 내용](https://blog.naver.com/ybook2006/221370126436)

---

## 포트 상태를 확인해봅니다.

![이미지](https://user-images.githubusercontent.com/43930419/88630179-c1525f80-d09f-11ea-9334-6ce886437daf.png)

```shell
sudo netstat
netstat
```

> netstat를 찾을 수 없다네요, 설치합니다.

![이미지](https://user-images.githubusercontent.com/43930419/88630182-c1eaf600-d09f-11ea-9b6a-f74cd9058611.png)

```shell
sudo apt install net-tools
```

> 포트 상태를 다시 확인합니다.
![이미지](https://user-images.githubusercontent.com/43930419/88630185-c1eaf600-d09f-11ea-82ba-212c9ac8b129.png)

```shell
netstat -tnlp
```

* `22`번 포트가 열려 있음을 확인할 수 있습니다.

---

## `ip`도 확인해 놓습니다.

![이미지](https://user-images.githubusercontent.com/43930419/88630187-c2838c80-d09f-11ea-8178-34bb326bad6e.png)

```shell
ifconfing
```

---

## ssh 접속용 계정을 만듭니다
> `jys-ssh`라고 명명해 주었습니다.

![이미지](https://user-images.githubusercontent.com/43930419/88630189-c31c2300-d09f-11ea-9faf-34f537946e40.png)

```shell
sudo adduser
```

> `adduser`로 만들었습니다.

### `adduser`와 `useradd`의 차이

* `adduser`는 패스워드 등등을 한번에 설정합니다.
* `useradd`는 `권한`을 일일히 다 설정해 주어야 합니다.

* [참고 블로그링크](https://mr-dan.tistory.com/6)


---

## putty를 이용해 접속해 봅니다.

* [아까 봐두었던](#`ip`도-확인해-놓습니다.) `ip`로 접속합니다.

![이미지](https://user-images.githubusercontent.com/43930419/88630191-c31c2300-d09f-11ea-8fd5-a7cbf7af16d6.png)

![이미지](https://user-images.githubusercontent.com/43930419/88630193-c3b4b980-d09f-11ea-9638-c5389f5227f1.png)

* `예` 를 눌러줍니다.


---

## backup 폴더를 만들고 권한을 변경해 줍니다.

* 폴더 생성 `mkdir`
* 권한 병경 `chmod`

![이미지](https://user-images.githubusercontent.com/43930419/88630195-c44d5000-d09f-11ea-94df-b6de00d8651c.png)

![이미지](https://user-images.githubusercontent.com/43930419/88630196-c44d5000-d09f-11ea-8f97-8401703333f0.png)

```shell
mkdir backup
chmod 764
```

### 764가 의미하는바
> `r` 실행, `w` 쓰기, `x` 실행

![이미지](https://user-images.githubusercontent.com/43930419/88630198-c4e5e680-d09f-11ea-992c-93031ae94c20.png)

![이미지](https://user-images.githubusercontent.com/43930419/88630199-c4e5e680-d09f-11ea-8f2e-1a070e965436.png)

* 소유자에게 `r` , `w` , `x` 권한을 준다 
    * `4 + 2 + 1` = `7`
* 그룹에게 `r` , `w` 권한을 준다
    * `4 + 2` = `6`
* 타인에게 `r` 권한을 준다.
    * `4` = `4`
* **아무런 권한을 주고 싶지 않다**
    * `0`

---

## 현재 날짜를 찍어봅니다
> date

```shell
date
```

![이미지](https://user-images.githubusercontent.com/43930419/88630201-c57e7d00-d09f-11ea-85ea-a479d8e0a3eb.png)

---

## 참고한 블로그들

* https://velog.io/@kys6879/%EC%9A%B0%EB%B6%84%ED%88%AC-18.04-SSH-%EC%84%A4%EC%B9%98
* https://antdev.tistory.com/48
* https://simplehanlab.github.io/ssh/ssh-how_to_setup_ssh/