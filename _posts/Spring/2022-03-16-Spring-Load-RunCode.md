---
title: CommandLineRunner를 이용해 스프링 초기 구동시 특정코드 실행하기
date: 2022-03-16-17:00
categories:
- Spring

tags:
- Spring
- CommandLineRunner
- ApllicatoinRunner

---

## 스프링 구동시 특정코드 실행하기
> `CommandLineRunner` 혹은 `ApplicationRunner` 인터페이스를 구현해줍니다.

---

## Goal
- 스프링 컨테이너 로드시 특정 코드들을 실행시키는 방법을 알아봅니다.
- **CammandLineRunner**와 **ApplicationRunner** 차이를 알아봅니다.
- 실행시켜야 하는 코드가 여러개인 경우, **순서**를 정하는법을 알아봅니다.

---

## CommandLineRunner
> 스프링이 로드될때 특정 코드를 실행할 수 있습니다.

- `CommandLineRunner`를 구현한 구현체는 컴포넌트 스캔이 되고, 스프링부트가 구동되는 시점에 `run()` 메소드를 실행하게 됩니다.
- 이때 run 메소드 안의 `...args`는 스프링 부트 구동시 실행 인자가 있다면, 받아서 처리하게 됩니다.

```java
import com.example.playground.domain.Member;
import com.example.playground.domain.repository.MemberRepository;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

@Component
public class DataLoader implements CommandLineRunner {
    private final MemberRepository memberRepository;

    public DataLoader(final MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

    @Override
    public void run(final String... args) throws Exception {
        memberRepository.save(new Member("unluckyjung"));   // 초기에 삽입할 데이터
    }
}
```

![image](https://user-images.githubusercontent.com/43930419/155511702-ae7b3766-3bbc-4efb-8e5b-00b7791fb0e2.png)

![image](https://user-images.githubusercontent.com/43930419/155511856-4bbfd950-68e0-4f5d-9ae0-5253e4110c49.png)

- 스프링 구동후 **unlcukyjung** 데이터가 DB에 삽입되어있는것을 확인할 수 있습니다.

### 순서를 정해서 로드하기
> `@Order(number)` 를 이용합니다.

```java
@Order(1)
@Component
public class DataLoader implements CommandLineRunner {
    ... 데이터 삽입 코드

    @Override
    public void run(final String... args) throws Exception {
        memberRepository.save(new Member("unluckyjung"));
    }
}


@Order(2)
@Component
public class DataLoader2 implements CommandLineRunner {
    ... 데이터 삽입 코드

    @Override
    public void run(final String... args) throws Exception {
        memberRepository.save(new Member("jys"));
    }
}
```

- 다음과 같이 `CommandLineRunner` 인터페이스를 구현한 컴포넌트가 여러개인 경우, `@Order`를 통해서 실행되는 순서를 정해줄 수 있습니다.

![image](https://user-images.githubusercontent.com/43930419/155517417-e030b7f5-38dc-469d-9c56-983f03bdb278.png)

![image](https://user-images.githubusercontent.com/43930419/155517496-d5709118-374c-45b4-805b-fa1dbebcd37e.png)

- unluckyjung **첫번째**, jys가 **두번째**로 데이터가 순차적으로 들어간것을 확인할 수 있습니다.

---

## ApplicationRunner
> String 형태가아닌 `ApplicationArguments` 타입으로 인자를 받아야되는경우 사용합니다.

> If you need access to ApplicationArguments instead of the raw String array consider using ApplicationRunner.  

- 스프링 [docs](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/CommandLineRunner.html)에 따르면 위와 같이 설명되어 있습니다.  
- 기본적으로 동작 자체는 `CommandLineRunner`와 완전히 동일하나, 받는 **인자 타입**만 다릅니다.


![image](https://user-images.githubusercontent.com/43930419/155519316-7d1afc5b-37f4-4798-a10b-fad76afe9c4f.png)

- `ApplicationArguments` 인터페이스 안을 들여다보면, 단순히 설정값과 같은 부분을 한단계 더 **포장해둔 상태**인것을 알 수 있습니다.

![image](https://user-images.githubusercontent.com/43930419/155520930-0b56fde9-cd41-4e1f-bc3f-4f6ee19b9222.png)

- 인자값 `unluckjung jys` 를 주어보겠습니다.

```java
@Order(3)
@Component
@Slf4j
public class DataLoader3 implements ApplicationRunner {
    @Override
    public void run(final ApplicationArguments args) throws Exception {
        for (int i = 0; i < args.getSourceArgs().length; ++i) {
            log.info("input" + i + ": " + args.getSourceArgs()[i]);
        }
    }
}
```


![image](https://user-images.githubusercontent.com/43930419/155520969-6723b120-c6a0-4c92-8c5c-a7f88dac4a59.png)

- 인자값이 공백기준 순차적으로 출력되는것을 로그를 통해 확인할 수 있습니다.


---

## Conclusion
- **CammandLineRunne**r와 **ApplicationRunner**를 통해 스프링 구동시 특정 코드들을 수행할 수 있다.
- 여러 인터페이스가 구현되어있는 경우 `@Order` 를 통해 실행 **순서**를 정해줄 수 있다.
- **ApplicationRunner**는 인자가 한단계 더 **포장**되어있는 형태일뿐이지, 동작자체는 CammandLineRunner와 동일하다.

---

## Code
- 해당 코드들은 전부 [github](https://github.com/unluckyjung/blog-codes/tree/main/spring-init-runner) 에서 보실 수 있습니다.

---




## Reference
- https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/CommandLineRunner.html
- https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/ApplicationRunner.html