---
title: Spring ThreadPoolTaskExecutor
date: 2022-08-27-18:00
categories:
- Spring
- Kotlin

tags:
- Spring
- Async
- ThreadPoolTaskExecutor

---

## Spring ThreadPoolTaskExecutor 에 대해서 정리해봅니다.
> 비동기를 위해 스프링에서 사용하는 스레드 풀을 설정할 수 있다.

---

## Goal
- Spring 에서 비동기 작업을 위한 스레드풀을 관리하는 `ThreadPoolTaskExecutor` 에 대해서 알아봅니다.
- 너무 많은 요청이 들어오는 경우에 대한 처리전략을 알아봅니다.
- Tomcat thread pool 과의 차이를 알아봅니다.


## threadPoolTaskExecutor 설정

```kotlin
import org.springframework.context.annotation.Bean
import org.springframework.context.annotation.Configuration
import org.springframework.scheduling.annotation.EnableAsync
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor
import java.util.concurrent.ThreadPoolExecutor

@Configuration
@EnableAsync
class AsyncConfig {

    @Bean
    fun threadPoolTaskExecutor(): ThreadPoolTaskExecutor {
        return ThreadPoolTaskExecutor().apply {
            this.corePoolSize = 5 // 기본 스레드 풀
            this.maxPoolSize = 20 // 최대 스레드 풀 (초과 요청 대기큐에도 다 찬 경우에 해당 사이즈로 증가해 작동)
            this.setQueueCapacity(50) // 초과 요청을 담는 큐의 개수
            this.setRejectedExecutionHandler(ThreadPoolExecutor.CallerRunsPolicy()) // 문제가 발생하는 경우 해당스레드에서 다시 처리 (CallerRunsPolicy)
            this.setWaitForTasksToCompleteOnShutdown(true) // shutdown 당해도 다른 작업 이어서 처리
            // this.setAwaitTerminationSeconds(60) // 어플리케이션 shutdown 후 최대 60초 대기 (6 까지는 처리하지 못한 요청들을 처리함)
            this.setThreadNamePrefix("unluckyjung task executor") // 접두사
        }
    }
}
```

- 명시적으로 `ThreadPoolTaskExecutor` 를 만들어주지 않는경우, `@Async` 어노테이션을 사용시 `SimpleAsyncTaskExecutor` 를 사용하게 되고, 비동기 호출마다 새로운 스레드를 만들게 됩니다.
- **즉 `ThreadPoolTaskExecutor` 를 명시적으로 만들어 주는것을 권장합니다.**

- 여기서 주의할점은 `maxPoolSize` 은 **기본 스레드 풀 + 초과 요청을 담는 큐의 개수** 를 넘는 요청이 들어온 경우에, 점진적으로 사이즈가 늘어나는 맥시멈을 정하는것입니다.
- 예를 들어 현재 옵션의 경우 `5(기본스레드 풀) + 50(초과 요청을 담는 큐의 개수)` 을 넘는 요청이 오는 경우 (ex 56개의 요청) 에 `maxPoolSize` 이 20까지 늘어나게 됩니다.

## 예외가 발생한 경우를 처리하는 전략
> 처리할 수 있는 큐의 용량을 넘는 경우에 대한 처리 전략을 정해줄 수 있습니다. 

```kotlin
this.setRejectedExecutionHandler(ThreadPoolExecutor.CallerRunsPolicy()) // 문제가 발생하는 경우 해당스레드에서 다시 처리 (CallerRunsPolicy)
```

- 위에서 만들어준 `threadPoolTaskExecutor` 의 경우에는 `20(max) + 50(대기 큐)`개의 요청까지 받을 수 있습니다.
- 만약 `71` 개의 요청이 오는 경우 문제가 발생하기 시작합니다.


### 전략 목록

- **AbortPolicy**: default 설정, `RejectedExecutionException` 발생 시킵니다. ( 서비스 쪽에서 `RejectedExecutionException` 에 대한 처리로직을 작성하지 않는다며, task 처리가 실패할 가능성이 있습니다.)
- **DiscardOldestPolicy**: 시간상 오래된 작업을 무시하고, 새로운 작업을 다시 처리하려고 시도합니다. **예외 조차도 발생하지 않습니다.** (task 처리가 실패할 가능성이 있습니다.)
- **DiscardPolicy**: 처리하는 작업을 무시합니다. **예외 조차도 발생하지 않습니다.** (task 처리가 실패할 가능성이 있습니다.)
- **CallerRunsPolicy**: 요청한 thread에서 직접 다시 처리합니다. 다만 shutdown(어플리케이션이 종료) 경우에는 제외합니다.


> 커스텀 전략


- 개발자가 직접 처리정책을 `RejectedExecutionHandler` 내의 `rejectedExecution` 을 구현하여 정해줍니다.

```kotlin
class MyRejectedExecutionPolicy : RejectedExecutionHandler {
    override fun rejectedExecution(r: Runnable?, executor: ThreadPoolExecutor?) {
        TODO("Not yet implemented")
    }
}
```

> Shutdown 처리

```kotlin
this.setWaitForTasksToCompleteOnShutdown(true)
```

- `threadPoolTaskExecutor` 에 해당 옵션을 준다면, 작업중인 task가 있는경우에는 어플리케이션이 shutdown이 되어도 작업이 같이 종료되지 않습니다.

---

## threadPoolTaskExecutor ThreadPool vs Tomcat ThreadPool
> 비동기 작업에 대한 처리 개수 vs 요청에 대한 처리 개수 

```kotlin
// tomcat
server.tomcat.max-threads = 300

vs

// threadPoolTaskExecutor
this.maxPoolSize = 20
```

- Tomcat의 의 경우에는 http 요청에 대해서 동시 요청을 처리할 수 있는 개수를 스레드 풀로 관리합니다.
- `threadPoolTaskExecutor`는 spring에서 **작업(task)** 을 비동기적으로 처리하기 위한 스레드의 개수를 정해둡니다.

- 만약 1개의 요청이 10개의 작업으로 나누어서 진행되어야 한다면, tomcat thread 는 1개만 필요할것이고, spring thread 는 10개가 필요할것입니다.

---

## Conclusion
- `ThreadPoolTaskExecutor` 빈 등록을 통해 스프링에서 사용하는 스레드 풀을 설정할 수 있다.
- `maxPoolSize` 는 대기큐까지 다 찬뒤에서야 적용되기 시작한다.
- 설정한 값을 넘어서는 너무 많은 요청이 오는 경우, 문제 처리에 대한 전략들을 정해줄 수 있다.

---

## Reference

### TaskExecutor
- https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/scheduling/concurrent/ExecutorConfigurationSupport.html#setRejectedExecutionHandler-java.util.concurrent.RejectedExecutionHandler-
- https://www.baeldung.com/java-rejectedexecutionhandler
- https://kapentaz.github.io/spring/Spring-ThreadPoolTaskExecutor-%EC%84%A4%EC%A0%95/#
- https://blog.outsider.ne.kr/1066
- https://kwonnam.pe.kr/wiki/springframework/async
- https://deep-dive-dev.tistory.com/11
- https://jongmin92.github.io/2019/03/31/Java/java-async-1/#ThreadPoolTaskExecutor

### vs Tomcat
- https://stackoverflow.com/questions/58077834/difference-between-spring-boot-threadpooltaskexecutor-and-server-tomcat-max-thre
- https://stackoverflow.com/questions/54126131/server-tomcat-max-threads-vs-corepoolsize-vs-spring-datasource-tomcat-max
