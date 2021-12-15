---
title: Gradle 버전을 Wrapper를 이용해 관리하기
date: 2021-12-15-23:00
categories:
- Dev

tags:
- Gradle
- Wrapper

---

## Gradle 버전을 Wrapper를 이용해 관리해봅니다.
> Gradle 빌드는 wrapper를 사용하는것이 권장됩니다.

---

## Goal
- Gradle Wrapper가 무엇인지 알아봅니다.
- Gradle Wrapper를 만들어봅니다.
- 프로젝트에서 관리하는 Gradle 버전을 변경해봅니다.

---

## What is Wrapper
- wrapper에 선언된 gradle 버전을 호출하여, 필요한 경우 미리 다운로드하는 스크립트 입니다.
- 다른 Gradle 버전이 설치되어 있는 환경에서도, 프로젝트를 문제없이 작동할 수 있습니다.
- 프로젝트를 표준화 할 수 있습니다.


![image](https://docs.gradle.org/current/userguide/img/wrapper-workflow.png)

- 워크플로우는 위와 같습니다.


---

## Wrapper 생성 및 Gradle 버전 명시

```
--  wrapper 생성  

gradle wrapper
```

- wrapper 파일이 생성됩니다.

```
-- wrapper 파일 확인

cd ./gradle/wrapper 

-- 생성된 properties 확인
cat gradle-wrapper.properties
```

- `./gradle/wrapper` 안에 `gradle-wrapper.properties` 라는 설정 파일이 생성된것을 확인 할 수 있습니다.


```
-- 버전 변경
./gradlew wrapper --gradle-version 변경할버전
./gradlew wrapper --gradle-version 7.3.1

-- 변경된 버전 재확인
cat ./gradle/wrapper/gradle-wrapper.properties
```

- 위의 명령을 통해 해당 프로젝트에서 사용할 gradle 버전을 정해줄 수 있습니다.
- 이후 `./gradle` 이 아닌 `./gradlew` 명령을 사용해 프로젝트에서 정한 gradle 버전으로 작업할 수 있게 됩니다.

---

## Conclusion
- gradle을 이용해 협업을 하는경우 Gradle Wrapper를 이용해 프로젝트 빌드 환경을 관리하는것이 좋다.
- 다른 환경에서도 프로젝트 버전 일관성이 유지되어 효율적이기 때문이다. (프로젝트가 환경에 종속되지 않는다)

---

## Reference
- [공식문서](https://docs.gradle.org/current/userguide/gradle_wrapper.html)
- https://park-jongseok.github.io/languages/java/2019/11/01/upgrade-gradle.html
- https://effectivesquid.tistory.com/entry/Gradle-%EB%B9%8C%EB%93%9C%EC%8B%9C%EC%8A%A4%ED%85%9C-%EA%B8%B0%EC%B4%88
- https://velog.io/@deepen/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-Gradle-%EB%B2%84%EC%A0%84-%EB%B3%80%EA%B2%BD%ED%95%98%EA%B8%B0