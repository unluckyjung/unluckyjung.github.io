---
title: Intellij 에서 Console Log 확장하기
date: 2022-10-29-18:00
categories:
- Tool

tags:
- IDE
- Intellij

---

## Intellij 에서 Console command history size 확장하기
> 설정값 에서 buffer size 를 확장해줍니다.

---

## Goal
- 많은 로그가 쌓이는 경우, 인텔리제이 콘솔창에서 로그가 누락되는 현상을 해결해 봅니다.

---

## 로그 버퍼 사이즈를 확장합니다.

> `command + ,` -> Editor -> General -> Console

![image](https://user-images.githubusercontent.com/43930419/198826912-a01000d9-c3ae-4c3a-85ab-07ddeacabd72.png)

- 기본적으로 디폴트 세팅은 사진과 같이 `300 size` 밖에 콘솔에 결과를 표시해주지 않습니다.
- 즉, 많은 로그가 출력되는 경우 디폴트 세팅으로는 로그가 누락될 가능성이 존재합니다.


![image](https://user-images.githubusercontent.com/43930419/196924994-69396911-bc74-4ac8-8309-96b6ec2a2e49.png)

- `Override console` 옵션 체크박스를 활성화 해주고, 적절한 크기를 기입해주면 됩니다. (최대 1024 KB)