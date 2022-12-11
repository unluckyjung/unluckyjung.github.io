---
title: Spring 비동기 테스트코드 작성법
date: 2022-12-11-18:10
categories:
- TestCode
- Kotlin

tags:
- Kotlin
- Spring
- TestCode
- Junit5

---

## 스프링 에서 비동기 로직을 테스트하는방법을 정리해봅니다. (Spring Async Logic TestCode)
> 비동기 로직을 대기시키거나, 테스트내에서 동기로 변환합니다.


## Goal
- 테스트코드 에서 비동기 로직 수행 결과를 기다려 테스트하는 하는 방법을 알아봅니다.
- 테스트코드 에서 비동기 로직을 동기로 바꾸어서 테스트하는 방법을 정리해봅니다.

---

## Config

### 테스트 어노테이션

```kotlin
@Target(AnnotationTarget.CLASS)
@Retention(AnnotationRetention.RUNTIME)
@TestConstructor(autowireMode = TestConstructor.AutowireMode.ALL)
annotation class TestEnvironment

@Target(AnnotationTarget.CLASS)
@Retention(AnnotationRetention.RUNTIME)
@Transactional
@SpringBootTest
@TestEnvironment
annotation class IntegrationTest
```

### 비동기 Config

```kotlin
@Configuration
@EnableAsync
class AsyncConfig : AsyncConfigurer {
    override fun getAsyncExecutor(): Executor {
        return ThreadPoolTaskExecutor().apply {
            this.corePoolSize = 5
            this.maxPoolSize = 20
            this.setQueueCapacity(50)
            this.setRejectedExecutionHandler(ThreadPoolExecutor.CallerRunsPolicy())
            this.setWaitForTasksToCompleteOnShutdown(true)
            this.setAwaitTerminationSeconds(60)
            this.setThreadNamePrefix("default task executor")
            this.initialize() // @Bean 어노테이션이 없어, 호출이 필수
        }
    }

    @Bean
    fun customThreadPoolTaskExecutor(): Executor {
        return ThreadPoolTaskExecutor().apply {
            this.corePoolSize = 5 // 
            this.maxPoolSize = 20 // 
            this.setQueueCapacity(50) 
            this.setRejectedExecutionHandler(ThreadPoolExecutor.CallerRunsPolicy())
            this.setWaitForTasksToCompleteOnShutdown(true) 
            this.setAwaitTerminationSeconds(60)
            this.setThreadNamePrefix("unluckyjung task executor")
        }
    }
}
```

## 비동기 로직이 섞인 테스트 예시
> 비동기 로직이 끝나기전에 검증을 진행하여 테스트가 실패합니다.

### 비동기 로직을 지닌 서비스

```kotlin
@Service
class DummyService {

    @Async("customThreadPoolTaskExecutor")
    fun asyncFun() {
        ObjectNumber.countUp()
        logger.info("this thread is ${Thread.currentThread().name}")
    }

    companion object {
        private val logger = LoggerFactory.getLogger(this::class.java)
    }
}

object ObjectNumber {
    var count: Int = 0
        private set

    fun countUp() {
        count++
    }
}
```

```kotlin
@IntegrationTest
class SpringAsyncTest(
    private val dummyService: DummyService,
) {
    @Test
    fun asyncTest() {
        dummyService.asyncFun()
        ObjectNumber.count shouldBe 1   // 위에서 진행되고 있는 비동기 로직이 끝나기전에 verify
    }
}
```

![image](https://user-images.githubusercontent.com/43930419/206840973-a5fd1786-26ff-49f5-ac9b-f67261195e6c.png)

- 위의 테스트 코드는 실패하는 테스트 코드입니다.
- 이유는 비동기 로직에서 `ObjectNumber` 안의 `count` 를 증가시키기전에 테스트코드내에서 검증을 하게되어 테스트가 실패하게 됩니다.
- 이를 해결해주기 위해서는 결과값을 검증하기전에, **비동기 로직을 반드시 수행시킨뒤에** 테스트에서 검증을 해야한다는 조건이 붙게 됩니다.


## 비동기 로직을 테스트코드에서 대기시켜서 처리

```kotlin
@IntegrationTest
class SpringAsyncTest(
    private val dummyService: DummyService,

    @Qualifier("customThreadPoolTaskExecutor")
    private val customThreadPoolTaskExecutor: TaskExecutor
) {
    @Test
    fun asyncTest() {
        dummyService.asyncFun()
        val executor = customThreadPoolTaskExecutor as ThreadPoolTaskExecutor
        executor.threadPoolExecutor.awaitTermination(1, TimeUnit.SECONDS)

        ObjectNumber.count shouldBe 1
    }
}
```

- 비동기 로직을 수행하는 `ThreadExecutor` 를 테스트코드 에서 DI 받게 합니다.
- 인터페이스 형태이므로, 실제 구현체인 `ThreadPoolTaskExecutor` 으로 타입변환 해줍니다. 
- (빈등록을 처음부터 `ThreadPoolTaskExecutor` 형태로 해두면 생략가능합니다.)
- 해당 스레드의 동작이 수행되기를 1초까지 기다린 후 검증을 진행합니다.
- `(1, TimeUnit.SECONDS)` 대기시간은 비동기 로직이 얼마나 걸리는지에 따라 적절히 조절하면되나, 일반적인 테스트코드에서는 1초내에 해결이 가능합니다.
- 위와 같은 방법으로 **비동기 로직을 테스트 코드에서 대기시킨 후** 검증하여 테스트 코드내에서 비동기 로직을 포함하여 로직을 테스트 해볼 수 있습니다.


## 비동기 로직을 동기로직으로 변화 시켜서 처리


### config

```yaml
-- application.yml

spring:
  main:
    allow-bean-definition-overriding: true
```

- 스프링부터 `2.1` 이상 부터는 빈 오버로딩이 기본적으로 불가능하게 되어있습니다.
- 이를 해결해주기 위해서 `application.yml` 에서 빈 오버로딩이 가능하도록 설정값을 줍니다.

```kotlin
@IntegrationTest
class SpringAsyncTest2(
    private val dummyService: DummyService,
) {
    @Import(KopringApplication::class)
    @Configuration
    internal class SyncTaskExecutorConfigForTest {
        @Bean("customThreadPoolTaskExecutor")
        fun customThreadPoolTaskExecutor(): TaskExecutor {
            return SyncTaskExecutor()
        }
    }

    @Test
    fun asyncTest() {
        dummyService.asyncFun()

        ObjectNumber.count shouldBe 1
    }
}
```

### 테스트 코드

![image](https://user-images.githubusercontent.com/43930419/206841722-48c463c9-569c-4e32-8abf-c61cdd974817.png)

- 테스트를 위해서 로드되는 어플리케이션 이름을 `@Import` 해줍니다. (비동기 관련 설정 (`AsyncConfig`)을 오버로딩 하기 위함)
- `inner class` 로 테스트에서 비동기로직에 사용되는 `TaskExecutor` 를 동기로 작동하는 `SyncTaskExecutor` 으로 변경하여 오버로딩 해줍니다. `@Bean("customThreadPoolTaskExecutor")`
- 위처럼 해주면 테스트 코드내에서는 **비동기 로직이 테스트 코드와 동기 로직으로 묶여** 테스트가 성공하게 됩니다.

> 참고: 오버로딩 하는 Config 을 테스트 클래스내 `inner class` 로 넣어주어야만 해당 테스트 클래스에서 Bean 을 오버로딩해줄 수 있습니다.
> TODO: Junit5 내 이너클래스 Config 에서의 빈 등록 순서 찾아보기 `2022.12.11`


### But 추천하지 않음

> 하지만 이 방법은 실제 로직은 비동기로 도는데, 테스트코드내에서 동기로 억지로 바꾸는 방법이라 실제 배포환경에서는 실패할 가능성이 존재합니다. 
> 실제로도 현업 업무중에 위와같은 테스트 코드를 작성해서 기능이 정상적으로 도는것으로 생각했으나, 비동기로직에서만 발생했던 트랜잭션 문제 떄문에 배포후에 에러가 나는 상황을 겪어본적이 있습니다.
> 가능하면 해당 방법보다는 **비동기 로직을 테스트코드에서 대기시켜서 처리** 테스트 하는방법을 권장드립니다.

---

## Conclusion
- 테스트 코드내에서 비동기 로직을 완료될떄까지 잠시 대기시켜, 비동기 로직을 포함한 테스트 코드를 작성할 수 있다.
- 테스트 코드내에서 비동기 로직을 테스트코드 스레드와 동기화 시켜, 테스트를 진행할 수 있으나 추천하진 않는다.


---

## Code
- 예시와 관련된 코드는 [Github](https://github.com/unluckyjung/kotlin-spring-jpa-playground/tree/test/spring-async-test) 에서 볼 수 있습니다.

---

## Reference
- https://stackoverflow.com/questions/42438862/junit-testing-a-spring-async-void-service-method
- https://www.baeldung.com/spring-boot-bean-definition-override-exception
- https://shashaka.github.io/springboot/2018/02/28/spring-async-method-test.html


