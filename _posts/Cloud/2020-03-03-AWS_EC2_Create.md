---
title: AWS EC2 서버 구축하고, 접속해보기
date: 2020-03-30-19:00
categories:
- Cloud

tags:
- AWS
- EC2

photos:
- https://upload.wikimedia.org/wikipedia/commons/thumb/9/93/Amazon_Web_Services_Logo.svg/200px-Amazon_Web_Services_Logo.svg.png

---

## AWS EC2 인스턴트를 생성해봅니다.
> 외부 접속 포트를 열어 접속까지 해봅니다.

---

### Cloud란?
> 네트워크을 통해 실물 컴퓨팅 리소스를 `종량 과금제`로 제공하는 서비스


### AWS란?
> Amazon Web Services

- `AWS (아마존 웹 서비스)`는 아마존닷컴의 클라우드 컴퓨팅 사업부다.
- 다른 웹 사이트나 클라이언트측 응용 프로그램에 대해 온라인 서비스를 제공하고 있다. 
- 개발자가 사용 가능한 기능을 제공하는 플랫폼을 제공하는 `IaaS`이다.
    - `IaaS`, `PaaS`, `SaaS`, 는 차후 정리할 예정.


### EC2란?
> Elastic Compute Cloud

- `독립된 컴퓨터`를 사용자에게 임대해주는 서비스이다.
- `Elastic` **유연한, 탄력적인** Compute Cloud이다.
    - 사용한 만큼만 비용을 지불한다.
    - 1분만에 인스턴스를 만들 수 있다.


### 인스턴스란?
> 가상 컴퓨팅 환경

---

#### EC2 인스턴트를 만든다.
> 원하는 운영체제를 선택하여 진행한다.

![EC2](/post_images/aws_0.PNG)

![EC2](/post_images/aws_1.PNG)

* 나의 경우 아마존 리눅스를 선택했다.

![EC2](/post_images/aws_2.PNG)

![EC2](/post_images/aws_3.PNG)

---

#### Key Pair 를 생성하여 저장한뒤 완료한다.
> 재발급이 불가능하다. 잘 보관해야한다.

![EC2](/post_images/aws_4.PNG)

![EC2](/post_images/aws_5.PNG)

---

#### 인스턴트가 작동하고 있는지 확인한다.
> 노란색으로 표시한 IPv4 의 주소가, 내가 접속할 주소이다.

![EC2](/post_images/aws_6.PNG)

![EC2](/post_images/aws_7.PNG)

---

#### Git Bash를 이용하여 접속한다.
> 과거에는 `putty`를 이용하여 주로 접속했는데, `Bash`가 더 편한것 같다.

![EC2](/post_images/aws_7.PNG)
![EC2](/post_images/aws_8.PNG)

* `pem file`의 권한을 `chmod 400` 으로 바꿔 주었다. 
* `ssh` 접속 방법은 다음과 같다.

> ssh -i `your pem files name` @ `your IPv4 adderss`

![EC2](/post_images/aws_9.PNG)


---

#### 포트를 열어준다.
> 보안그룹에서 인바운드 규칙을 추가해준다.

![EC2](/post_images/aws_10.PNG)
![EC2](/post_images/aws_11.PNG)
![EC2](/post_images/aws_12.PNG)

* `3693` 포트로 접속시 `Hello EC2 instance!` 를 출력하는 소스를 작성해 두었다.
    * `express`, `node.js` 를 이용하였다.


#### 외부에서 주소로 접속해본다
> 성공!

![EC2](/post_images/aws_13.PNG)
![EC2](/post_images/aws_14.PNG)

---

> 좀 주구난방으로 적었는데, 나중에 시간 여유가 많을때 다듬는걸로 하고...

---
> Reference

[위키피디아](https://ko.wikipedia.org/wiki/%EC%95%84%EB%A7%88%EC%A1%B4_%EC%9B%B9_%EC%84%9C%EB%B9%84%EC%8A%A4)  
