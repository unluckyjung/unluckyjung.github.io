---
title: 인텔리제이에서 java 파일을 docs로 읽을때 해결방법
date: 2021-01-14-23:00
categories:
- Tool

tags:
- IDE
- Intellij

---

## 인텔리제이에서 프로젝트를 import 해왔을때, 파일을 docs로 잘못 읽는 상황을 해결해봅니다.
> Custom Poperties에서 파일 최대 크기를 늘려줍니다.

---

## 인텔리제이가 java 소스코드 파일을 docs로 읽는다?
> File size exceeds configured limit (2560000), code insight features not available

* 처음에 시도 해본 방법
    * File > invalid Caches / Restart 를 누른다
    * 나의 경우에는 이 방법으로 해결이 되지 않았다.

* 구글링을 통해 [해결책](https://stackoverflow.com/questions/23057988/file-size-exceeds-configured-limit-2560000-code-insight-features-not-availabl)을 찾았다.
    * **원인** : 인텔리제이에서 열 수 있는 **소스 코드 크기의 한도**를 넘어서서 자동으로 DOC로 설정된 것이었다.
    * **해결** : 열 수 있는 소스 코드 크기의 한도를 늘려준다.

---

## 해결방법

### 설치 버전
* `Help > Edit Custom Properties` 로 간다.
* `idea.max.intellisense.filesize=99999`를 추가해준다.


### 무설치 버전
* 인텔리제이 설치 위치의 `bin/idea.properties` 를 연다.
* `idea.max.intellisense.filesize=99999`를 추가해준다.


---

## 참고

* MAC 환경에서 인텔리제이 힙 메모리 늘리기
    * [링크](https://stackoverflow.com/questions/13578062/how-to-increase-ide-memory-limit-in-intellij-idea-on-mac/13581526#13581526) 참고.

* 인텔리 제이 `2020.3.1` 이상의 버전에서 memory indicator 표시하기
    * `View 탭 > Apperance > Status Bar Widgets > Memory Indicator`