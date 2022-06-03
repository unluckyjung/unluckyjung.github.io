---
title: Kotlin Paramaterized Test
date: 2022-06-03-13:00
categories:
- Kotlin
- Test

tags:
- Kotlin
- Test
- Junit5
- Kotest

---

## Kotlin Paramaterized Test
> Kotest, Junit5 두개를 기준으로 설명합니다.

---

## Goal
- Kotest 에서 **Paramaterized Test** 하는 방법을 알아봅니다.
- Junit5 으로 **Paramaterized Test** 하는 방법을 알아봅니다.

---

## Kotest

### forAll을 이용합니다.
> [Behavior Spec](https://kotest.io/docs/framework/testing-styles.html) 기준으로 작성하였습니다. 


```kotlin
import io.kotest.core.spec.style.BehaviorSpec
import io.kotest.data.blocking.forAll
import io.kotest.data.row
import io.kotest.inspectors.forAll
import io.kotest.matchers.shouldBe

class SampleKtTest : BehaviorSpec({
    given("4개의 숫자가 주어지고") {
        `when`("3개의 숫자를 더하면") {
            then("마지막 숫자의 값이 나온다.") {
                forAll(
                    row(1, 2, 3, 6)
                ) { a, b, c, d ->
                    a + b + c shouldBe d
                }
            }
        }
    }

    given("2개의 숫자가 더해지고") {
        fun sum(num1: Int, num2: Int) = num1 + num2
        `when`("두개를 sum 함수를 거치면") {
            then("더했을때의 결과가 나온다.") {
                forAll(
                    row(1, 2),
                    row(3, 4),
                    row(5, 6)
                ) { num1, num2 ->
                    sum(num1, num2) shouldBe num1 + num2
                }
            }
        }
    }

    given("1개의 숫자가 더해지고") {
        val nums = listOf(1, 2, 3, 4)
        `when`("자기 자신을 더하면") {
            then("2배 곱한결과가 나온다.") {
                nums.forAll {
                    it + it shouldBe it * 2
                }
            }
        }
    }
})
```

```kotlin
forAll(
    row(input1, input2, result),    // case 1
    row(input3, input4, result),    // case 2
) { paramNameA, paramNameB ->
    paramNameA + paramNameA shouldBe result
}
```

- `forAll` 와 `row` 를 이용하여 여러 파라메터와, 케이스를 넣어줄 수 있습니다.


---

## Junit5


### 의존성 추가

```gradle
testImplementation("org.junit.jupiter:junit-jupiter-params:5.8.2")
```

### 기본으로 제공해주는 타입을 넣어주기
> `@ValueSource`, `CsvSource` 를 이용합니다.

```kotlin
import io.kotest.matchers.shouldBe
import org.junit.jupiter.params.ParameterizedTest
import org.junit.jupiter.params.provider.ValueSource

class JunitSampleTest {

    @ParameterizedTest
    @ValueSource(ints = [1, 2, 3, 4, 5])
    fun sumTest(num: Int) {
        sum(num, 3) shouldBe num + 3
    }

    @ParameterizedTest
    @CsvSource("1,2", "3,4", "5,6")
    fun sumTest2(num1: Int, num2: Int) {
        sum(num1, num2) shouldBe num1 + num2
    }

    private fun sum(num1: Int, num2: Int): Int {
        return num1 + num2
    }
}
```

- `@ValueSource` 를 이용해 같은 타입의 값을 여러번 넣어줄 수 있습니다. (코틀린의 경우에는 `[]` 를 사용합니다.)
- `@CsvSource` 를 이용해 여러개의 값을 넣어줄 수 있습니다.


---

### 객체 타입을 넣어주기
> `@MethodSource` 를 이용합니다.


```kotlin
@ParameterizedTest
@MethodSource("memberCompareTest")  // "memberCompareTest" 생략가능
fun memberCompareTest(member1: Member, member2: Member) {
    member1 shouldBe member2
}

companion object {
    @JvmStatic
    fun memberCompareTest() = listOf(
        Arguments.of(Member("goodall"), Member("goodall")),
        Arguments.of(Member("unluckyjung"), Member("unluckyjung")),
    )
}
```

- 테스트 함수 이름과 object 내부의 함수이름이 같다면, `@MethodSource` 에 호출할 함수 이름을 생략해주어도 됩니다.
- `@MethodSource` 의 경우 `@JvmStatic` 을 이용하여 처리해주어야 합니다.
  - `@MethodSource`는 내부적으론 자바기반으로 작성되어있고 static method를 호출하여 매개변수를 생성합니다. 
  - 하지만 Java에서는 kotlin의 object의 메서드를 static 변수처럼 `ClassName.methodName()` 와 같은 방식으로 접근을 못하기 때문에, `@JvmStatic` 이 필요합니다.


---

## Reference
- https://www.baeldung.com/kotlin/junit-5-kotlin
- https://kotlinlang.org/docs/reference/annotations.html