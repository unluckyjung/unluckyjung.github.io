---
title: 윈도우 10 파워쉘 한글 설정
date: 2020-12-02-12:35
categories: 
- ETC

tags:
- Window10
- PowerShell

---

## 파워쉘 환경에서 한글깨짐을 해결해봅니다.
> profile 세팅을 해줍니다.

## 한글이 안나와!
> git cli 를 복습중인데 git log를 찍을시 한글이 전부 깨져서 나오는 상황 발생

---

## profile을 설정해줍니다.

<br>

![](/post_images/powerShell_Korean_troubleShooting_0.jpg)

```console
$profile
```

* 파워쉘을 키고 `$profile` 을 입력해줍니다.
* 아래 나오는 경로로 갑니다.
* `Document` 다음에 경로가 없을 수 있습니다.
    * 그러면 폴더와 파일을 **직접 만들어줍니다.**
* 파워쉘을 껏다 켜봅니다.

---

## 에러가 발생한다면?

<br>

![](/post_images/powerShell_Korean_troubleShooting_1.jpg)

* 새로운 에러가 나왔네요.

![](/post_images/powerShell_Korean_troubleShooting_2.jpg)

* 침착하게 구글링을 해봅니다.

![](/post_images/powerShell_Korean_troubleShooting_2.5.jpg)

* [글에 적힌대로](https://stackoverflow.com/questions/4037939/powershell-says-execution-of-scripts-is-disabled-on-this-system) 따라서 해봅니다.
    * **관리자** 모드로 파워쉘을 킵니다.
    * `Set-Set-ExecutionPolicy RemoteSigned` 을 입력합니다.

```console
Set-ExecutionPolicy RemoteSigned
```

---

## 결과

<br>

![](/post_images/powerShell_Korean_troubleShooting_3.jpg)

* 에러가 사라졌고, 이후에 한글도 잘 출력되는것을 작업을 통해 확인할 수 있습니다.

---

## Reference
* [StackOverflow](https://stackoverflow.com/questions/4037939/powershell-says-execution-of-scripts-is-disabled-on-this-system)