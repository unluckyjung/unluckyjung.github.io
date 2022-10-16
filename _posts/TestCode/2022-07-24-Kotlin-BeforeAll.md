---
title: Kotlin + Junit5 에서 BeforeAll, AfterAll 사용하기
date: 2022-07-24-14:30
categories:
- TestCode
- Kotlin

tags:
- Kotlin
- TestCode
- Junit5

---

## Kotlin 에서 @BeforeAll, @AfterAll 을 사용해봅니다.
> 정적 메소드를 사용하거나, 테스트 생명주기를 조절합니다.


---

## Goal
- Kotlin 에서 Junit5에서 제공하는 `@BeforeAll`, `@AfterAll` 을 이용해 최초 1회만 동작하고, 테스트별로 매번 동작하지 않게 하는 방법 2가지를 알아봅니다.

---

## companion object + @JvmStatic
> `@JvmStatic` 를 이용해 자바의 제공하는 static 메소드와 동일한 동작을 하게 해줍니다.


```kotlin
companion object {
    var count = 0;

    @BeforeAll
    @JvmStatic
    internal fun setUp() {
        count++
    }
}
```

- 일반적으로 `@BeforeAll` 와 같은 클래스에서 1번만 호출이 보장되어야 하는 어노테이션은, java 버전의 정적 메서드를 호출해야합니다.
- 이를 위해서 `@BeforeAll` 을 companion object 에 넣어주고, `@JvmStatic` 를 붙여 줍니다.


### Code

```kotlin
import io.kotest.matchers.shouldBe
import org.junit.jupiter.api.BeforeAll
import org.junit.jupiter.api.Test
import org.junit.jupiter.api.TestInstance

class JunitBeforeAllTest {

    @Test
    fun test1() {
        count shouldBe 1
    }

    @Test
    fun test2() {
        count shouldBe 1
    }

    companion object {
        var count = 0;

        @BeforeAll
        @JvmStatic
        internal fun setUp() {
            count++
        }
    }
}
```

---

## 테스트의 생명 주기를 조절합니다.
> 클래스 단위의 생명주기로 설정합니다.

```kotlin
@TestInstance(TestInstance.Lifecycle.PER_CLASS) // 생명 주기 조절, 디폴트값은 PER_METHOD
```

- 기본적으로 디폴트값은 메서드 단위로 생명주기가 잡혀있기 떄문에, `@BeforeAll` 과 같은 어노테이션을 사용하기 위해서는 static 메소드를 호출시켜야 합니다.
- 하지만 Junit5에 와서는 테스트 생명주기를 `@TestInstance.Lifecycle.옵션)` 을 이용해 클래스 단위로도 조절할 수 있습니다.
- 즉, 위와 같은 방법을 이용하면 코틀린에서 굳이 ``@JvmStatic` 을 사용해 static 메소드를 만들지 않고도 `@BeforeAll` 과 같은 클래스단위의 호출이 가능해집니다.

### Code

```kotlin
@TestInstance(TestInstance.Lifecycle.PER_CLASS) // 생명 주기 조절, 디폴트값은 PER_METHOD
class JunitBeforeAllTest2 {
    var count = 0;

    @BeforeAll
    internal fun setUp() {
        count++
    }

    @Test
    fun test1() {
        count shouldBe 1
    }

    @Test
    fun test2() {
        count shouldBe 1
    }
}
```

---

## Conclusion
- `Companion object` + `@JvmStatic` 를 사용해 static 메소드를 이용한 `@BeforeAll` 호출이 가능하다.
- Junit5에서는 `@TestInstance.Lifecycle.옵션)` 를 이용해 정적메서드를 이용하지 않고도 `@BeforeAll` 사용이 가능하다.

---

## Reference
- https://www.baeldung.com/java-beforeall-afterall-non-static
- https://www.baeldung.com/junit-testinstance-annotation

