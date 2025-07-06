---
title: Kotlin 에서 우아하게 Logger 사용하기
date: 2025-07-06-14:00
categories:
- Kotlin

tags:
- Kotlin

---

## 확장 함수로 로거 생성시 발생하는 보일러플레이트 코드를 제거해봅니다.
> **확장 함수(Extension Function)**와 Reified Generic을 활용하여 로거(Logger) 생성을 간소화 해봅니다.

---

## Goal
- `private val logger = LoggerFactory.getLogger(MyClass::class.java)` 와 같이 반복적으로 작성하는 코드를 Kotlin의 확장 함수를 이용해 간결하게 만들 수 있는법을 정리해봅니다.

---

## 선결론

```kotlin
inline fun <reified T> T.logger(): Logger = LoggerFactory.getLogger(T::class.java)

class MyAwesomeService {
    // ... 비즈니스 로직 ...

    companion object {
        private val logger = logger()   // 보일러 플레이트가 제거된 한줄
    }
}
```

- 작성한 유틸리티 클래스를 사용하여 사용하여 단 한 줄로 클래스에 로거를 추가할 수 있게 됩니다.

## 코드 설명

```kotlin
import org.slf4j.Logger
import org.slf4j.LoggerFactory

inline fun <reified T> T.logger(): Logger = LoggerFactory.getLogger(T::class.java)
```


- `fun <reified T> T.logger()`: 모든 타입 T에 대해 logger()라는 확장 함수를 추가합니다.
- `inline`: 함수를 호출하는 곳에 함수 본문 코드를 그대로 복사해 넣습니다. 이는 reified 키워드를 사용하기 위한 전제 조건입니다.
- `reified`: inline 함수 내에서 제네릭 타입 T의 실제 타입을 런타임에 알 수 있게 해줍니다. 덕분에 `T::class.java`처럼 실제 클래스 정보를 가져와 LoggerFactory에 전달할 수 있습니다. companion object 내에서 함수를 호출하더라도 컴파일러가 감싸고 있는 외부 클래스의 타입을 정확히 추론하여 로거를 생성해줍니다.


## 테스트 코드로 동작 검증

```kotlin
import ch.qos.logback.classic.Level.ERROR
import ch.qos.logback.classic.Level.INFO
import ch.qos.logback.classic.Logger
import ch.qos.logback.classic.spi.ILoggingEvent
import ch.qos.logback.core.read.ListAppender
import io.kotest.matchers.shouldBe
import org.junit.jupiter.api.DisplayName
import org.junit.jupiter.api.Test
import org.slf4j.LoggerFactory


internal class MyLoggerTestDummyClass {

    fun infoLog(message: String) {
        logger.info("$message info log")
    }

    fun errorLog(message: String) {
        logger.error("$message error log")
    }

    companion object {
        private val logger = logger()
    }
}


class LoggerFactoryTest {

    @DisplayName("MyLoggerFactory 안의 logger 로 Info 로그를 찍히면 로그 히스토리가 남는다.")
    @Test
    fun testInfoLogger() {
        val logger = LoggerFactory.getLogger(MyLoggerTestDummyClass::class.java) as Logger
        val listAppender = ListAppender<ILoggingEvent>()
        listAppender.start()
        logger.addAppender(listAppender)

        val dummyClass = MyLoggerTestDummyClass()

        dummyClass.infoLog("yoonsung")
        listAppender.list.filter {
            (it.level == INFO) && (it.message == "yoonsung info log")
        }.size shouldBe 1

        listAppender.stop()
    }

    @DisplayName("MyLoggerFactory 안의 logger 로 error 로그를 찍히면 로그 히스토리가 남는다.")
    @Test
    fun testErrorLogger() {
        val logger = LoggerFactory.getLogger(MyLoggerTestDummyClass::class.java) as Logger
        val listAppender = ListAppender<ILoggingEvent>()
        listAppender.start()
        logger.addAppender(listAppender)

        val dummyClass = MyLoggerTestDummyClass()

        dummyClass.errorLog("unluckyjung")

        listAppender.list.filter {
            (it.level == ERROR) && (it.message == "unluckyjung error log")
        }.size shouldBe 1

        listAppender.stop()
    }
}
```

- `listAppender` 를 이용해 의도했던 로그가 남았는지를 확인하였습니다.

---

## Conclusion
- Kotlin의 확장 함수와 reified 제네릭을 활용하면, 단 한 줄의 유틸리티 함수로 프로젝트 전체의 로거 생성 코드를 개선할 수 있다. 

---

## Reference
- https://kotlinlang.org/docs/inline-functions.html#reified-type-parameters