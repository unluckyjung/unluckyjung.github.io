---
title: Kotlin 에서 여러 조건을 가지고 distinct 하기
date: 2023-04-16-14:00
categories:
- Kotlin

tags:
- Kotlin

---

## Kotlin 에서 여러 조건을 가지고 distinct 하는 방법을 알아봅니다.
> 객체내 여러 필드를 가지고 조건을 따져 중복된 결과를 걸러내봅니다.

---

## Goal
- distinct 를 여러조건으로 하는방법을 알아봅니다.

---

## 테스트할 객체

```kotlin
data class Man(
    val name: String = "yoonsung",
    val nickName: String = "goodall",
    val githubName: String = "unluckyjung",
    val age: Int = 31,
)
```

---

## 2개의 조건을 테스트
> Pair 를 이용합니다.


```kotlin
@DisplayName("중복처리 조건이 2개인 경우, Pair 를 이용하면 필터링된다.")
@Test
fun twoConditionDistinctTest() {
    val sameMan = Man()
    val sameNameAndNickName = Man(githubName = "yoonsung")
    val diffMan = Man(name = "goodall")

    val list = listOf(sameMan, sameMan, sameNameAndNickName, diffMan)

    val distinctByNameAndNickName = list.distinctBy {
        Pair(it.name, it.nickName)
    }

    distinctByNameAndNickName.size shouldBe 2
    distinctByNameAndNickName.shouldContainAll(sameMan, diffMan)
}
```

- 만약 `name` 과 `NickName` 이 모두 같은 사람들은 중복처리한다고 생각해보겠습니다.
- 이경우에는 `list.distinctBy {Pair(it.name, it.nickName)}` 와 같이, 중복된 것으로 처리할것들을 Pair 로 묶어서 넣어주면됩니다.


### distinct 는 Selector 로 처리됩니다.

```kotlin
@DisplayName("같은 이름과 닉네임을 가진것들은 1개로 처리된다. (distinct 는 Selector 사용에 주의)")
@Test
fun twoConditionDistinctTest2() {
    val man = Man() // sameNameAndNickName
    val diffName = Man(name = "a")
    val diffNickName = Man(nickName = "b")
    val diffGithubName = Man(githubName = "c") // sameNameAndNickName
    val diffAge = Man(age = 10) // sameNameAndNickName

    val list = listOf(man, diffName, diffNickName, man, diffGithubName, diffAge, man)

    // name 과 nickName 이 같은것은 하나로 처리
    val distinctByNameAndNickName = list.distinctBy {
        listOf(it.name, it.nickName)
    }

    distinctByNameAndNickName.size shouldBe 3
}
```

- `diffAge` 객체의 경우에는 다른 Age 를 가지고 있으나, 중복된 Man 으로 처리된것을 볼 수 있습니다.
- 왜냐하면 `distinctBy` 안에 들어가는것은 **selector** 로, 들어간 필드들의 값이 같으면 중복 처리를 하기 떄문입니다.
- 즉 `distinctBy` 안에 조건식을 넣어서는 안됩니다. `(ex nickName == "goodall")`

### 결과로 남는 객체의 순서
> list 순서상 앞에 있던 객체가 남습니다.

```kotlin
@DisplayName("중복조건이 처리되는경우, list 상 앞에있는 객체가 우선으로 남는다.")
@Test
fun distinctOrderTest() {
    val sameNameMan1 = Man(age = 30)
    val sameNameMan2 = Man(age = 20)

    val list = listOf(sameNameMan1, sameNameMan2)

    val distinctByName = list.distinctBy {
        it.name
    }

    distinctByName.size shouldBe 1
    distinctByName.shouldContain(sameNameMan1)
}
```

- `age = 30` 이 객체가 list 안에 순서상 먼저 들어가있었기 때문에, `age = 20` 가 제거된것을 확인할 수 있습니다.

---

## 3개 이상의 조건을 필터링 하는 경우
> `listOf(field1, field2, field3)`

```kotlin
@DisplayName("중복 조건이 3개인 경우, distinctBy 에 list 를 이용하면된다.")
@Test
fun threeConditionDistinctTest() {
    val sameMan = Man()
    val diffAgeMan = Man(age = 10)

    val list = listOf(sameMan, sameMan, sameMan, diffAgeMan)

    val distinctByNames = list.distinctBy {
        listOf(it.name, it.githubName, it.nickName)
    }

    // name 에 관련된 조건 3개가 전부 같음.
    distinctByNames.size shouldBe 1
    distinctByNames.shouldContainAll(sameMan)


    // age 는 달라서 2개로 구분되어짐.
    val distinctByNames2 = list.distinctBy {
        listOf(it.name, it.githubName, it.nickName, it.age)
    }
    distinctByNames2.size shouldBe 2
    distinctByNames2.shouldContainAll(sameMan, diffAgeMan)
}
```

- pair 로 처리될 수 없는 3개이상의 경우, `Triple(3개 인경우)` , `listOf()` 로 처리해주면됩니다.


---

## Conclusion
- `distinctBy` 를 이용해 N개의 필드값 을 따져서 중복을 제거할 수 있다.
- 비교할 필드가 2개인경우, `Pair`, 그 이상의 경우에는 `listOf` 를 이용해 처리할 수 있다.

---

## Reference
- https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/distinct-by.html
- https://stackoverflow.com/questions/45883719/how-can-i-remove-duplicate-objects-with-distinctby-from-a-list-in-kotlin