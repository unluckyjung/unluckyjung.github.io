---
title: Kotest Testing Styles 코드를 인텔리제이를 이용하여 실행하기
date: 2022-04-04-00:07
categories:
- TestCode
- Kotlin

tags:
- Kotlin
- TestCode
- Junit5
- IntlliJ
- Kotest

---

## Kotest Testing Styles 코드를 인텔리제이에서 실행하기
> kotest plugin을 설치해주어야 합니다.


---

## Goal
- 코틀린에서 제공하는 여러가지 스타일 코드를 `@Test` 어노테이션 없이 구동해봅니다.

---

## 왜 안될까?
> 인텔리제이에서 테스트코드 구동이 안되는 상황

![image](https://user-images.githubusercontent.com/43930419/160280757-84fa1672-52a0-471a-9d5c-d0ecda29b9db.png)


![image](https://user-images.githubusercontent.com/43930419/160280704-b93bb1e5-91e0-4749-9260-5442b58a78ea.png)

- [kotest 공식문서](https://kotest.io/docs/framework/testing-styles.html)에서 **여러가지의 스타일 spec**을 제공하길래, 한번 써보고자 따라서 코드를 작성해봤지만 도대체 코드 실행을 **어떻게 하는지를 알수없는 상황**에 처했습니다.
- 위의 사진을 보면 해당 테스트를 수행할 수 있는 방법이 보이지 인텔리제이 상에서 보이지 않습니다.



![image](https://user-images.githubusercontent.com/43930419/160281196-7ae1c278-4f74-4529-8abf-e382a3b44376.png)

- `@Test` 어노테이션 기반은 그냥 기존과 같은 방식으로 인텔리제이에서 테스트 수행이 가능합니다.
- 하지만, 위와 같이 시로 **BDD 스타일**이 적용된 코드는 해당 클래스내에선 아예 실행하는방법을 못찾겠어가지고 삽질을 하기 시작했습니다.

![image](https://user-images.githubusercontent.com/43930419/160280687-ba90ed71-70ea-48f4-8499-97527c42bf78.png)

- 우형기술블로그에 소개된것을 보아도, 그냥 따로 코드상에 차이는 없었습니다.


- 구동방법에 따른 방법을 [공식문서](https://kotlinlang.org/docs/jvm-test-using-junit.html#run-a-test)에서 찾아봐도, 단순히 어노테이션 기반으로만 설명이 되어있어서 더욱 미궁에 빠지기 시작했습니다.

---

## Kotest 플러그인을 설치합니다.
> [코틀린지기(진)](https://youtu.be/ewBri47JWII) 의 제이슨의 구제

![image](https://user-images.githubusercontent.com/43930419/160281211-d8731ed3-8805-4794-9a6e-aadf251bd90b.png)

- 도저히 모르겠어가지고, 코틀린 권위자 제이슨에게 질문을 했습니다.

![image](https://user-images.githubusercontent.com/43930419/160280680-1b26a9b5-9844-47b5-a858-576023735a3c.png)

![image](https://user-images.githubusercontent.com/43930419/160280698-0e34a381-2e41-4678-8a2b-c0c23d3d415d.png)

- **kotest puglin** 을 설치하고 난뒤, 9시, 10시방향에 코드를 수행할 수 있는 **실행 버튼**이 나타났습니다.

---

## Conclusion
- 코틀린에서 제공하는 다양한 스타일을 인텔리제이에서 구동하려면 **kotest 플러그인**을 설치해야 한다.
- 별거 아닌 부분인거 같은데, 고민하는 시간이 길어지면 그냥 질문을 하자.

