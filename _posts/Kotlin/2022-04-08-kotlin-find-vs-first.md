---
title: Kotlin find vs first
date: 2022-05-01-20:30
categories:
- Kotlin

tags:
- Kotlin
- Collections

---

## 찾는 값이 없는경우, 처리하는 동작이 다릅니다.
> find는 null을 반환, first는 예외가 발생 합니다.

---

## Goal
- kotlin **collections** 에서의 find와 first 차이를 알아봅니다.

---

## find
> 조건에 일치하는 값을 찾지 못한경우 null을 반환합니다.

```kotlin
enum class Rank(val matchCount: Int) {
    RANK1(3),
    RANK2(2),
    RANK3(1),
    FAIL(0);

    companion object {
        fun getRankByMatchCount(count: Int): Rank {
            return values().find() {
                it.matchCount == count
            } ?: FAIL
        }
    }
}
```

- find는 찾지 못한경우 null을 반환하기 때문에, 엘비스 연산자 `:?` 로 처리해서 디폴트 값을 정해 내보내 줄 수 있습니다.


```kotlin
fun getRankByMatchCount(count: Int): Rank? {
    return values().find() {
        it.matchCount == count
    }
}
```

- 엘비스 연산 처리를 하지 않는 경우, null이 될 수 있으므로 반환값이 `Rank?` 가 됩니다.

## first
> 조건에 일치하는 값을 찾지 못한경우 예외가 발생합니다.

```kotlin
fun main() {
    println(Rank.getRankByMatchCount(-1))
}

enum class Rank(val matchCount: Int) {
    RANK1(3),
    RANK2(2),
    RANK3(1),
    FAIL(0);
    
    companion object {
        fun getRankByMatchCount(count: Int): Rank {
            return values().first() {
                it.matchCount == count  // 찾지못해서 예외발생!
            }
        }
    }
}
```

```java
Exception in thread "main" java.util.NoSuchElementException: Array contains no element matching the predicate.
```

- first의 경우 찾은 값들 중에서 가장 앞의 값을 내보내게 되는데, `-1` 은 Enum중 매칭되는것이 없으므로 null이 아니라 **예외가 발생하게 됩니다.**

```kotlin
fun main() {
    println(Rank.getRankByMatchCount(-1))   // null
}

enum class Rank(val matchCount: Int) {

    ...

    companion object {
        fun getRankByMatchCount(count: Int): Rank? {
            return values().firstOrNull() {
                it.matchCount == count
            }
        }
    }
}
```

- 만약 찾는값이 없는경우, 예외가 아닌 null 값을 내보내고 싶다면 `findOrNull()` 을 사용해주면 됩니다.

---

## Conclusion
- 찾는값이 없는경우 find는 Null 반환, first는 예외를 발생시킵니다.
- 예외발생을 원치 않는 경우 `firstOrNull` 로 바꾸면 null이 반환됩니다.
- 저의 경우에는 find가 더 직관적이여서 find를 사용할것 같으나, **java** 의 stream 처럼 [병렬 처리 상황에서의 차이](https://codechacha.com/ko/java8-stream-difference-findany-findfirst/#findfirst-vs-findany)가 있는지를 이후에 좀 더 찾아보아야 겠습니다. `05.01`


---


## Reference
- https://stackoverflow.com/questions/45520267/what-is-the-difference-between-find-and-firstornull
- https://youtrack.jetbrains.com/issue/KT-5185
- https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/first-or-null.html
- https://codechacha.com/ko/java8-stream-difference-findany-findfirst/#findfirst-vs-findany