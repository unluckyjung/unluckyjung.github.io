---
title: Spring + Junit5에서, 의존성 주입을 @AutoWired로 받아야 하는 이유
date: 2021-08-21-00:00
categories:
- TestCode

tags:
- TDD
- TestCode
- Junit5
- Spring

---

## Junit5 에서, 생성자로 빈을 주입 받을경우 왜 문제를 일으킬까?
> `org.junit.jupiter.api.extension.ParameterResolutionException` 에러가 왜 발생할까?

---

## Goal
- Junit5에서 의존성 주입을 생성자 주입으로 할경우, 문제가 생기는 원인을 찾아봅니다.

---

## 개요
```
org.junit.jupiter.api.extension.ParameterResolutionException: 
No ParameterResolver registered for parameter 
```

- 태스트 코드에서 생성자를 통해서 의존성을 주입하니 다음과 같은 에러가 발생하였다.
- **jupiter** 가 ParameterResolver 를 찾지 못했다고?

---

## Jupiter는 스프링 빈을 직접 가져오지 못한다.
> 환경에 따른 주체의 차이. Jupiter vs Spring

- **결론부터 이야기하자면 Junit5의 경우 빈 주입은 `@AutoWired` 어노테이션을 사용해야 한다.**
- 테스트 프레임워크(Junit5) 에서 주체는 Spring이 아닌, Jupiter가 되므로 `@Autowired`를 명시적으로 선언해주어야 **Jupiter**가 **Spring Container**에게 빈 주입을 요청할 수 있게 된다.

- 스프링 프레임워크의 경우 **Spring Ioc 컨테이너**가 등록할 **Bean** 들을 먼저 찾아서 보관하고 있다.
  - 이후 생성자 주입을 요구하는경우, 적절한 Bean을 찾아서 생성자 주입을 수행하게 된다.
  
- 테스트 프레임 워크의 경우 생성자 매개 변수 관리를 **Jupiter**가 하게된다.
- 생성자 주입을 요구하는경우, 생성자 매개변수를 처리할 **ParameterResolver**을 열심히 뒤져보게 되지만, 해당 빈은 스프링이 가지고 있기때문에 처리하지 못하게된다.
  - 이 때문에 **나 못찾았어!** 하고, 에러가 발생하게 된다.
- 하지만 `@AutoWired` 어노테이션을 달아 명시해 주게 된다면, **Jupiter**가 빈 주입을 **스프링 컨테이너**에게 요청하게 되어서, 정상적으로 빈 주입을 받을 수 있게된다.

---

## Conclusion
- 테스트 환경, 프로덕트 환경은 관리하고 있는 프레임워크 주체가 다르다.
- Junit5에서는 **Jupiter**가 주체이기 떄문에, 생성자를 통해 빈을 주입받을 수 없다.

---

## Reference
- https://sunghs.tistory.com/138
- https://minkukjo.github.io/framework/2020/06/28/JUnit-23/
- https://nipafx.dev/junit-5-architecture-jupiter/#JUnit-5