---
title: 인텔리제이 한글 깨짐, project not found in root project 해결법
date: 2020-12-08-19:00
categories:
- Tool

tags:
- IDE
- Intellij

photos: 
- /post_images/gradle_JVM_Error1.jpg

---

## 인텔리제이에서 한글이 계속 깨져서 나오는 것을 해결해 봅니다.
> Gradle JVM 이 원인 이었습니다.

---

## 문제 발생 원인
> 인텔리제이 `20.3` 설치

* 인텔리제이 `2020.3`이 새로나왔길래 노트북에 다운 받아봤습니다. [링크](https://www.jetbrains.com/ko-kr/idea/whatsnew/)
    * 데스크탑에서 진행중인 프로젝트를 `Github` 에 올리고, 노트북에서 다시 내려 내려 받았습니다.

## 문제 발생
> 한글이 깨져서 나오기 시작합니다.

* 처음에 이문제를 해결 하기 위해서, [이전글](https://unluckyjung.github.io/tool/2020/12/02/Intellij-UTF8-setting/) 을 참고해서 똑같이 시도했습니다.
    * 하지만 해결이 되지 않았습니다.
    * `재빌드` 와 캐시를 다시 지워봐도 동일한 이슈가 계속 발생했습니다.

## 문제 원인 추측
* 혹시나 하고, 새 프로젝트를 생성하고 빌드를 해보니 **문제 없이 작동합니다.**
    * 문제가 발생하는 프로젝트를 `Run`이 아닌 `Build` 를 해봅니다.
    > project not found in root project` 라는 에러를 만나게 됩니다.  

    * 사실 처음부터 Run도 안되었어야 하는거 같은데 Run은 왜 되었는지 의문입니다.
    * 뭔가 빌드 부분에 문제가 생겼을것이라고 추측을 하게 됩니다.

---

## 문제 해결
![](/post_images/gradle_JVM_Error.jpg)
![](/post_images/gradle_JVM_Error1.jpg)

* `Gradle` 의 `JVM` 이 이상하게 선택되어 있었습니다.=
* 이부분을 제가 사용하는 `JDK1.8` 로 맞춰서 변경해 주었습니다.
    * 빌드가 정상적으로 되고, 한글이 문제 없이 출력 되었습니다.