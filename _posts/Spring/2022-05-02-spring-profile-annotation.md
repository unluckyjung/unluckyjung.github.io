---
title: Profile 어노테이션을 이용해 테스트 수행을 프로필 별로 구분하기
date: 2022-05-02-00:10
categories:
- Spring

tags:
- Spring
- Java
- Annotation
- Profile


---

## @Profile로 테스트 수행을 프로필 별로 구분하기
> @ActiveProfile 을 같이 사용하여, 테스트 코드 수행시 빈을 구분하여 로드할 수 있습니다.

## Goal
- `@Profile` 어노테이션을 이용해 프로필을 구분하여 빈을 로드해봅니다.
- `@ActiveProfile` 어노테이션을 이용해 테스트 수행시 특정 빈만 로드해봅니다.

---

## @Profile, @ActiveProfile
> 스프링 3.1 이상부터 사용할 수 있는 기능입니다.


```java
@Component
@Profile("test")
public class SampleBean () {
  return new foo();
}

@Bean
@Profile("!test")
public class RealService () {
  return new bar();
}
```
- `@Profile` 어노테이션을 이용해 빈이나, 컴포넌트에게 프로필을 정해줄 수 있습니다.
- 이때 부정의 `!` 연산을 사용할 수 있습니다. 위와 같은경우 test 프로필이 아닌 경우에만 `RealService()` 가 활성화 됩니다.


```java
@ActiveProfiles("test")
public class AcceptanceTest {
    @LocalServerPort
    int port;

    @BeforeEach
    public void setUp() {
        RestAssured.port = port;
    }
}
```

- 테스트 코드 수행시 `@ActiveProfile` 을 이용해 특정 프로필의 빈만 로드하면서 테스트를 수행할 수 있습니다.
- **AcceptanceTest**를 수행하면 `RealService()` 는 빈으로 등록되지 않고, **SampleBean()** 만 빈으로 등록 됩니다.

### 옵션을 여러개 주기
> 여러 프로필을 등록할 수 있습니다.

```java
@Bean
@Profile({"!dev", "!prod"})
public class testService testService() {
  return new testService();
}
```

- 중괄호를 이용해 N개의 프로필 정보를 넣어줄 수 있습니다.

---

## 스프링 .yml 설정에서 프로필 옵션을 넣어주기
> profile 별로 설정을 분리 할 수 있습니다.

### 여러개의 .yml 파일로 구분하기
![image](https://user-images.githubusercontent.com/43930419/160855274-3d3792cc-0972-48e4-b467-7c4608e8bd01.png)

- 과거 진행했던 [babble 프로젝트](https://github.com/woowacourse-teams/2021-babble/tree/develop/back/babble/src/main/resources)의 `yml` 파일들입니다.
- 위와같이 `application-{profileName}.yml` 과 같은 형식으로 프로필을 구분해서 `yml` 파일을 만들어줄 수 있습니다.

### 한개의 .yml 파일에서 구분하기
> `---` 구분자를 이용해 프로필별 로 구분할 수 있습니다.


```yml
spring:
  profiles:
    active: local

---

spring:
  profiles: local

server:
  port: 8080

---

spring:
  profiles: test

server:
  port: 8081

---

spring:
  profiles: prod

server:
  port: 8082

```

- `yml` 파일을 한개만 만들어두고, **local**, **test**, **prod** 3개의 프로필을 정해두고 구분자 `---` 를 적어 구분할 수 있습니다.
- 위의 `yml` 설정의 경우 default 옵션으로는 **local** 프로필로 동작하게 됩니다.



```console
$ java -jar -Dspring.profiles.active={profileName} {jarFileName}

// local 프로필로 구동
$ java -jar -Dspring.profiles.active=local babble.jar

// prod 프로필로 구동
$ java -jar -Dspring.profiles.active=prod babble.jar
```

- `cli` 에서는 다음과 같이 프로필을 선택하여 **jar** 파일을 실행할 수 있습니다.



---

## Conclusion
- `@Profile`을 이용하여 프로필 설정을 해두고, 테스트코드에서 `@ActiveProfile`을 이용해 특정 빈만 로드하거나, 로드하지 않을 수 있다.
- `.properties` 나 `.yml` 파일에서 프로필을 구분하여 설정값을 다르게 줄 수 있다.

---

## Reference
- https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/env/Profiles.html
- https://www.baeldung.com/spring-profiles
- https://galid1.tistory.com/664