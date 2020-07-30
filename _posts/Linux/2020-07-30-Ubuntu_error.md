---
title: Vmware Player and Device/Credential Guard are not compatible 해결법
date: 2020-07-30-15:00
categories:
- Linux

tags:
- linux
- ubuntu
- error

photos: 
- https://user-images.githubusercontent.com/43930419/88628916-23aa6080-d09e-11ea-8f6b-e4c2100a8156.png

---

## ubuntu 설치중 발생한 에러를 해결해봅니다.
> `Vmware Player and Device/Credential Guard are not compatible`

---

## 오류 원인
> 가상화 옵션

![이미지](https://user-images.githubusercontent.com/43930419/88628918-2442f700-d09e-11ea-9c32-b98aecb67e1e.png)
* 저의 경우 `docker`를 사용한적이 있어서, `Hyper-V` 옵션이 켜져 있었습니다.

## 오류 해결
> `window 10` 기준

![이미지](https://user-images.githubusercontent.com/43930419/88628921-2442f700-d09e-11ea-81ea-3c46b87dda2f.png)
![이미지](https://user-images.githubusercontent.com/43930419/88628923-24db8d80-d09e-11ea-9e97-0180f154ab2c.png)

* `winodws 기능 켜기/끄기` 로 들어가서 `Hyper-V` 옵션을 꺼줍니다.

---