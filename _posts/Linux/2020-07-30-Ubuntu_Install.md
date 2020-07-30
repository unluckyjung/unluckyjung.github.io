---
title: ubuntu 18.04 설치 (VMware 14)
date: 2020-07-30-16:00
categories:
- Linux

tags:
- linux
- ubuntu

photos: 
- https://user-images.githubusercontent.com/43930419/88628944-2907ab00-d09e-11ea-96ef-b9b4735cb487.png

---

## `VMware 14`에 `Ubuntu 18.04` 를 설치 해 봅니다.
> `desktop` 버전을 오랜만에 설치해 봅니다.

---

## 우분투 `18.04` 설치

## 이미지 파일 다운로드
> [링크](http://mirror.kakao.com/ubuntu-releases/bionic/)

![설치페이지](https://user-images.githubusercontent.com/43930419/88628889-1f7e4300-d09e-11ea-837d-a14bc09bec0e.png)

* 우분트 이미지 파일 `iso` 을 다운 받습니다.

## VMware 에서 설치
![이미지](https://user-images.githubusercontent.com/43930419/88628897-20af7000-d09e-11ea-9b76-8a512d8114af.png)
* `Create a New VM` 을 클릭합니다.

![이미지](https://user-images.githubusercontent.com/43930419/88628898-20af7000-d09e-11ea-8335-6e1421f654b6.png)
![이미지](https://user-images.githubusercontent.com/43930419/88628899-21480680-d09e-11ea-9199-fbe102c85136.png)

* OS는 `Linux`, 버전은 `Ubuntu`를 선택합니다. 

![이미지](https://user-images.githubusercontent.com/43930419/88628901-21480680-d09e-11ea-915d-d27ba1bb2df3.png)
![이미지](https://user-images.githubusercontent.com/43930419/88628902-21e09d00-d09e-11ea-910a-6fe0728f4741.png)

* 경로를 설정하고, 디스크 크기를 정해 줍니다.

![이미지](https://user-images.githubusercontent.com/43930419/88628904-21e09d00-d09e-11ea-8130-271299684c56.png)

* `Customizise Hardware`를 클릭 합니다.

![이미지](https://user-images.githubusercontent.com/43930419/88628905-22793380-d09e-11ea-9126-7c32f7249713.png)

* 저의 경우, `CPU`가 `8코어`에  `RAM`이 `32G` 이므로 넉넉히 잡았습니다.
    * `CPU 4` , `RAM 8` 

![이미지](https://user-images.githubusercontent.com/43930419/88628907-22793380-d09e-11ea-9f2f-15d277e9affb.png)
![이미지](https://user-images.githubusercontent.com/43930419/88628910-2311ca00-d09e-11ea-9d61-f229862ba95b.png)

* [위에서 설치한](#이미지-파일-다운로드) 이미지 파일을 불러옵니다.

![이미지](https://user-images.githubusercontent.com/43930419/88628911-2311ca00-d09e-11ea-8779-7efeb241ea5d.png)

![이미지](https://user-images.githubusercontent.com/43930419/88628914-23aa6080-d09e-11ea-8838-652f93a4b0d8.png)

* 설치된것을 확인하고 :arrow_forward: 를 눌러서 실행을 시도해봅니다.

![이미지](https://user-images.githubusercontent.com/43930419/88628916-23aa6080-d09e-11ea-8f6b-e4c2100a8156.png)

* `???`
* 원인을 찾아보니, `가상화 옵션`이 켜져있어서 해당 현상이 발생한다고 합니다.
* [링크](https://unluckyjung.github.io/Linux/2020/07/30/Ubuntu_error)를 참고하여 해결합니다.

---


## 우분투 설치

![이미지](https://user-images.githubusercontent.com/43930419/88628924-24db8d80-d09e-11ea-81f0-13ded4103206.png)

* 설치가 시작 됩니다.

![이미지](https://user-images.githubusercontent.com/43930419/88628926-25742400-d09e-11ea-9227-766bc673d302.png)

* 학부시절 교수님이 해주셨던 말을 기억하며 `영어` 를 선택 합니다.

![이미지](https://user-images.githubusercontent.com/43930419/88628927-260cba80-d09e-11ea-85d6-17dedee5f225.png)

* 키보드는 `한글` 이니 레이아웃을 한글에 맞춥니다.

![이미지](https://user-images.githubusercontent.com/43930419/88628927-260cba80-d09e-11ea-85d6-17dedee5f225.png)
![이미지](https://user-images.githubusercontent.com/43930419/88628930-260cba80-d09e-11ea-9267-a00b91fe8949.png)
![이미지](https://user-images.githubusercontent.com/43930419/88628934-273de780-d09e-11ea-9b6c-52ec8610af23.png)

![이미지](https://user-images.githubusercontent.com/43930419/88628935-273de780-d09e-11ea-8cd2-252534078cb5.png)

* 열심히 `continue` 버튼을 눌러 줍니다.

![이미지](https://user-images.githubusercontent.com/43930419/88628937-27d67e00-d09e-11ea-8c0b-7fa8c1c75b70.png)

* 계정과 암호를 설정합니다.

![이미지](https://user-images.githubusercontent.com/43930419/88628938-27d67e00-d09e-11ea-84af-9fe379a98aff.png)
![이미지](https://user-images.githubusercontent.com/43930419/88628940-286f1480-d09e-11ea-947e-5d1005c8516a.png)
![이미지](https://user-images.githubusercontent.com/43930419/88628943-286f1480-d09e-11ea-9447-c22b9317b9ba.png)

* 시키는대로 재부팅을 합니다.

---

## 해상도 변경

![이미지](https://user-images.githubusercontent.com/43930419/88628944-2907ab00-d09e-11ea-96ef-b9b4735cb487.png)

* [링크](https://unluckyjung.github.io/Linux/2020/07/30/Ubuntu_display)를 참고해 해상도를 변경해 봅니다.

---

## 연관 링크

* [우분투 설치](https://unluckyjung.github.io/Linux/2020/07/30/Ubuntu_install)
* [가상화 옵션으로 인한 설치 에러 해결](https://unluckyjung.github.io/Linux/2020/07/30/Ubuntu_error)
* [해상도 변경](https://unluckyjung.github.io/Linux/2020/07/30/Ubuntu_display)
