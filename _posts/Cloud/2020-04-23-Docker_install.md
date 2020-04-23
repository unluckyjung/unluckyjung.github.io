---
title: 윈도우 10 도커 설치 후 우분투 이미지 만들기
date: 2020-04-23-10:46
categories:
- Cloud

tags:
- 가상화
- Docker

photos: 
- /post_images/docker-logo.png

---

## 도커를 윈도우 10에 설치합니다.
> 우분투 이미지를 다운로드 하고 컨테이너를 만들어봅니다.

---

### 도커란 무엇일까?
> Docker `부두 노동자` 컨테이너를 다루는 것

* 도커는 이미지를 실행하고, 이미지를 저장 배포 한다.
* 도커는 서비스 운영 환경을 묶어 손쉽게 **배포**하고 **실행**하는 `경량 컨테이너 기술`
* 도커는 `VM` 이 아니다.
* 도커는 `가상화`를 하는것이 아니다.
* 컨테이너에 가상 공간을 만들지만, 실행 파일을 **호스트에서 직접 실행한다.**
* 컨테이너 기술은 가상화가 아닌 **프로세스를 격리** 하는것이다.

### 가상화 기술
* `VM`은 완전한 컴퓨터를 설치한다.
* 하이퍼 바이저 위에서 `게스트 OS`가 생성된다.
* 응답속도를 개선하기 위해서 호스트와 커널을 공유하는 `반 가상화` 기술이 생겼다.

#### 가상화 기술의 한계
* 아무리 최적화를 해도 `게스트 OS`가 필요하고, 오버헤드로 인한 **성능 문제** 를 극복하지 못한다.
* 배포, 관리 기능도 많이 부족하다.

### 도커의 특징

* 하이퍼 바이저 위치에 `도커 엔진`이 있다.
    * 하지만 그 위에 `게스트 OS`가 필요 없다.
* 필요한 것만 설치한다 (용량 확보)
* 호스트와 `System call`을 공유한다.
    * 호스트와 컨테이너 사이의 성능차이가 미미하다.
* 이미지 생성, 배포가 아주 용이하다.
    * 베이스 이미지에서 바뀐 부분만 이미지로 만든다.
    * 다른곳에서 바뀐 부분만 합쳐서 실행 시키면 된다. 

---

## 도커 설치 준비


![가상화 옵션 확인](/post_images/docker_-1.PNG)
![가상화 옵션 확인](/post_images/docker_-2.PNG)

* 가상화 옵션이 켜있는지 확인합니다.
* 켜져 있지 않다면 
    * [VT](https://support.bluestacks.com/hc/ko/articles/115003174386-PC%EC%97%90%EC%84%9C-%EA%B0%80%EC%83%81-VT-%ED%99%9C%EC%84%B1%ED%99%94%ED%95%98%EA%B8%B0)
    * [Hyper-X](https://docs.microsoft.com/ko-kr/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v)
    * 두 링크를 참고하여, 사전준비를 마쳐줍니다.


<br><br>

## 도커 다운로드 설치


![도커 다운로드](/post_images/docker_0.PNG)
![도커 다운로드](/post_images/docker_1.PNG)

* 도커를 [공식 사이트](https://docs.docker.com/docker-for-windows/install/#download-docker-for-windows) 에서 다운로드 합니다.


<br><br>

![도커 설치](/post_images/docker_3.PNG)
![도커 아이콘](/post_images/docker_4.PNG)

* 도커를 설치하고 재부팅 후 도커를 실행시킵니다.

<br><br>

![파워쉘](/post_images/docker_5.PNG)
* `Power shell`에서 `docker -v`를 입력하여 도커가 잘 설치되었는지 확인합니다.

---

## 우분투 이미지 다운, 실행

![우분투 이미지 다운](/post_images/docker_6.PNG)
* **pull**
* `docker pull ubuntu` 를 입력해 최신 ubuntu를 다운 받습니다.
    * `tag`를 설정하지 않으면 디폴트로 `:latest` 버전이 다운 됩니다.

<br><br>

![우분투 이미지 확인](/post_images/docker_7.PNG)

* `docker images` 를 입력해 설치된 이미지를 확인합니다.

<br><br>


---

## 컨테이너 생성

![우분투 컨테이너 생성](/post_images/docker_8.PNG)
* **run**
* `docker run -i -t --name HelloDocker ubuntu /bin/bash` 를 입력합니다.
    * `ubuntu /bin/bash` 우분투 이미지 안의 bin/bash 를 실행합니다.
    * `-i -t` **interactive**, **Pseudo-tty**, 실행된 Bash에 입출력을 합니다.
    * `--name HelloDocker` 컨테이너의 이름을 **HelloDocker**로 설정 합니다.

<br><br>

![우분투 컨테이너 작업](/post_images/docker_9.PNG)
* 컨테이너 내부에서 `home`에다가 `Yoonsung`이라는 디렉토리를 만들어 보고 확인해 봅니다.

<br><br>

---

## 컨테이너 종료, 실행, 접속

![우분투 컨테이너 종료](/post_images/docker_10.PNG)
* **exit**
* `exit` 을 입력해 컨테이너를 **종료**합니다.
* **ps -a**
* `docker ps -a`를 입력해 현재 **설치된** 컨테이너 목록을 봅니다.

<br><br>


![우분투 컨테이너 실행](/post_images/docker_11.PNG)
* **start**
* `docker start HelloDocker`를 입력해 **HelloDocker** 컨테이너를 실행시킵니다.
* 자기의 이름이 리턴되며 실행이 된것을 알 수 있습니다.

<br><br>

![우분투 컨테이너 접근](/post_images/docker_11-1.PNG)
![우분투 컨테이너 접근](/post_images/docker_12.PNG)
* **attach**
* `docker attach HelloDocker`을 입력해 아까 실행시킨 컨테이너에 **접속** 합니다.
* 아까 만든 `Yoonsung` 디렉토리가 있는것을 확인 할 수 있습니다.

<br><br>

---

## 백그라운드

![우분투 컨테이너 종료](/post_images/docker_13.PNG)
* `ctrl + p ` , `ctrl + q`를 차례대로 입력하고 컨테이너를 빠져나옵니다.

<br><br>

---

![실행중 컨테이너 확인](/post_images/docker_14.PNG)
* **ps**
* `docker ps` 를 입력해 현재 **실행중인** 컨테이너를 확인해 봅니다.

<br><br>

![컨테이너 안 명령 실행](/post_images/docker_15.PNG)
* **exec**
* `exec 컨테이너이름 명령 매개변수` 를 입력합니다.
* `docker exec HelloDocker ehco "Hello World"`
    * HelloDocker 컨테이너는 응답해라 "Hello World 라고
* **Hello World** 라고 응답한것을 볼 수 있습니다.

---

## 컨테이너 종료

![컨테이너 종료](/post_images/docker_16.PNG)
* **stop**
* `docker stop HelloDocker` 를 입력해 **HelloDocker** 컨테이너를 종료합니다.
* `docker ps` 를 통해 확인하면 실행중인 컨테이거가 없는것을 확인 할 수 있습니다.

---

## 참고사항
* 지금 노트북에서는 괜찮은데 데스크탑에서는 Docker명령어 반응에 로딩이 있다
    * 이유와 해결방법을 찾아봐야겠다.

---

## Reference
* https://www.slideshare.net/pyrasis/docker-fordummies-44424016
* https://www.popit.kr/%EA%B0%9C%EB%B0%9C%EC%9E%90%EA%B0%80-%EC%B2%98%EC%9D%8C-docker-%EC%A0%91%ED%95%A0%EB%95%8C-%EC%98%A4%EB%8A%94-%EB%A9%98%EB%B6%95-%EB%AA%87%EA%B0%80%EC%A7%80/
* https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html