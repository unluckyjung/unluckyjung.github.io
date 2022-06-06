---
title: Kotlin을 Java로 디컴파일(Decompile) 하기
date: 2022-06-06-13:00
categories:
- Kotlin
- IntelliJ

tags:
- Kotlin
- IntelliJ

---

## Kotlin으로 작성된 코드를 Java로 디컴파일(Decompile) 하기
> IntelliJ 의 기능을 이용합니다.

---

## Goal
- IntelliJ 의 기능을 이용하여 Kotlin으로 작성된 코드를 Java로는 어떻게 작성되었는지 확인해봅니다.

---

## IntelliJ Tool 옵션
> `Tools -> Kotlin -> show Kotlin Bytecode`

![image](https://user-images.githubusercontent.com/43930419/171986811-48766102-844a-4e1d-bd76-a767b75eedfd.png)

- **12시** 방향의 `Tools -> Kotlin -> show Kotlin Bytecode` 을 클릭합니다.

![image](https://user-images.githubusercontent.com/43930419/172131826-e9772e01-578b-40bd-ba4f-9f39a7dd5733.png)

- 혹은 `shift` 를 두번 입력 후, **kotlin byte** 와 같은 키워드를 입력해서 창을 띄울수도 있습니다.


![image](https://user-images.githubusercontent.com/43930419/171986822-53d41bfb-6399-4322-a180-33b0e44ca775.png)

- 해당 창에서는 바이트 코드를 확인할 수 있습니다.
- 브라우징된 우측창에서 11시 방향의 `Decompile` 버튼을 누릅니다.

![image](https://user-images.githubusercontent.com/43930419/171986836-40a0b12a-73cf-4b2a-b778-d36b0aefc647.png)

- 자바로 변환된 코드를 확인할 수 있습니다.


---

## Conclusion
- kotlin으로 작성한 코드를 intellij의 Tool을 이용하여 java 버전으로 확인할 수 있습니다.
- 이 방법을 통해서, 자바에 익숙한 개발자가 코틀린으로 코드를 작성할때 의도한 방향대로 코드를 작성했는지 확인하며, 사이드 이펙트를 줄여나갈수 있습니다

---

## Reference
- https://1minute-before6pm.tistory.com/28
