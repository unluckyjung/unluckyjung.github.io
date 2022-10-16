---
title: Kotlin Collections vs Sequence
date: 2022-04-08-20:30
categories:
- Kotlin

tags:
- Kotlin
- Sequence
- Collections

---

## Kotlin Collections vs Sequence 연산차이
> Eager로 처리하느냐 Lazy하게 처리하느냐의 차이가 있습니다.

---

## Goal
- Collection과 Sequence의 연산과정의 차이점을 알아봅니다.

---

## 컬렉션 연산의 차이 (Eager vs Lazy)
> 중간 연산 결과를 바로 생성해 두는가, 생성하지 않고 최종연산때 순차적으로 생성하는가


### Collection
> eager

- **collections**의 map과 filter와 같은 중간연산은 결과를 **즉시 생성**해둡니다.
- **map -> filter** 와 같이 연쇄적인 작업이 있는경우, map에 대한 중간연산을 거치고 중간 결과를 새로운 컬렉션에 미리 담아두게 됩니다.

```kotlin
val people = listOf(Person("jys", 29), Person("goodall", 30))

// kotlin collection
people.map {
    println("call map")
    it.age
}.filter {
    println("call filter $it")
    it >= 30
}.toList()
```

- 즉 위와 같은 코드가 있는경우 **map에 대한 연산을 전부다 마친뒤(2회) 결과를 저장해두고**, 이어서 filter를 처리하게 됩니다.
- 연산 횟수 N 만큼의 저장공간이 더 필요해지게 됩니다. 즉 예시의 경우에는 2개의 원소밖에 없어서 상관이 없지만, 원소가 많은 케이스에는 사용에 주의해야합니다.



### Sequence
> lazy

- `asSequence()` 를 이용하는 경우에는 컬렉션을 시퀀스로 변경하여 중간 연산 결과를 **저장하지 않게 됩니다.**

```kotlin
val people = listOf(Person("jys", 29), Person("goodall", 30))

people.asSequence().map {
    println("call map")
    it.age
}.filter {
    println("call filter $it")
    it >= 30
}.toList()
```



- `toList()` 를 호출하는 순간에서야, 순차적으로 연쇄 작업이 진행되게 됩니다.
- 이 형태는 **java의 stream**과 굉장히 유사한 형태임을 알 수 있습니다.

```kotlin
println("start")

// kotlin asSequence
people.asSequence().map {
    println("call map")
    it.age
}.filter {
    println("call filter $it")
    it >= 30
}//.toList()

println("end")
```

```console
start
end
```

- `toList()` 를 주석처리한다면, 중간연산에 들어간 출력문에서 **아무런 출력이 되지 않는것**을 볼 수 있습니다.
- `collections` 의 경우에는 `toList()`가 없어도 **call map, call filter** 가 출력이 됩니다.

---

## Sample Code

```kotlin
fun main() {
    val people = listOf(Person("jys", 29), Person("goodall", 30))

    // kotlin collections
    people.map {
        println("call map")
        it.age
    }.filter {
        println("call filter $it")
        it >= 30
    }.toList()

    println("=====================")

    // kotlin asSequence
    people.asSequence().map {
        println("call map")
        it.age
    }.filter {
        println("call filter $it")
        it >= 30
    }.toList()
}

class Person constructor(
    val name: String,
    val age: Int
)
```

```console
call map
call map
call filter 29
call filter 30
=====================
call map
call filter 29
call map
call filter 30
```

---

## Conclusion
- 원소가 많은 컬렉션에 대한 연산이 연쇄적으로 있는경우 시퀀스를 사용하는것이 좋다.

---

## Reference
- [kotlin docs](https://kotlinlang.org/docs/sequences.html)
- [코틀린 인 액션](http://www.yes24.com/Product/Goods/55148593)
- https://codechacha.com/ko/kotlin-sequences/
