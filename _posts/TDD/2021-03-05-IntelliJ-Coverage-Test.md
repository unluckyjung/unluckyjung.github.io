---
title: IntelliJ Coverage Test
date: 2021-03-05-15:00
categories:
- TDD

tags:
- Java
- IntelliJ
- TDD

---

## 인텔리제이에서 테스트 코드의 커버범위를 확인해봅니다.
> Intellij Coverage Test를 이용합니다.

- 인텔리제이 버전 : `20.3.1` 

---


## 과연 내 테스트 코드가 많은 부분을 커버하고 있을까?
> 내가 작성한 메소드, 상태값들을 제대로 테스트하고 있을까?

- 조건, 예외상황에 따른 테스트는 직접 테스트를 하는것이 맞다.
- 하지만 특정 메소드에 대해서 테스트 자체를 빼먹는 경우도 종종 발생한다.
  - 이부분을 IDE의 도움을 받아 보완하자!

---

## 인텔리제이에서 제공하는 기능을 이용해보자.
> 기억하세요 **방패**모양

### 방법1
![image](https://user-images.githubusercontent.com/43930419/110073720-34088380-7dc3-11eb-9ffb-8eb157706662.png)

- 우측 상단을 보면 방패 모양의 아이콘이 있다.
- 해당 아이콘을 클릭하면 테스트 코드를 실행시키며, 테스트 코드가 어디까지 커버하고 있는지를 확인할 수 있게된다.

### 방법2
![image](https://user-images.githubusercontent.com/43930419/110074180-1be53400-7dc4-11eb-8a17-97f050a3cd69.png)

- 12시 방향의 탭 메뉴중 `Run`을 클릭후, `Test in ... with Coverage ` 을 눌러 테스트를 진행 시킨다.

### 방법3
![image](https://user-images.githubusercontent.com/43930419/110075503-3c15f280-7dc6-11eb-90cb-c6a2bdfd46aa.png)

- 테스트 코드파일(...java) 로 가서 `Run .... with Coverage` 을 눌러 테스트를 진행 시킨다.

---

## 결과

![image](https://user-images.githubusercontent.com/43930419/110074591-ce1cfb80-7dc4-11eb-815b-14e5f0c35f7a.png)

- 지금 테스트 코드가 프로덕션 코드의 어디까지를 커버하고 있는지를 퍼센테이지로 확인할 수 있게 된다.

---

## Reference
- https://www.jetbrains.com/help/idea/running-test-with-coverage.html
- https://soft.plusblog.co.kr/77