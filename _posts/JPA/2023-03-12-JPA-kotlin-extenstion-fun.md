---
title: JPA + Kotlin 사용시 확장함수 활용법
date: 2023-03-12-17:00
categories:
- JPA
- Kotlin

tags:
- JPA
- Kotlin

---

## JPA + Kotlin 사용시 확장함수 활용법
> 조회성 함수를 확장해서 사용할 수 있습니다.

---


## Goal
- JPA + Kotlin 사용시 확장함수 활용법을 알아봅니다.
- id 를 기반해 조회할시, 옵셔널이나 null 이 아닌 예외 or 리턴값을 가지도록 확장함수를 작성해봅니다.

---

## 선 결론
> 확장함수를 이용해, id를 통해 엔티티를 찾는 공통함수를 작성해줄 수 있습니다.

```kotlin
inline fun <reified T, ID> CrudRepository<T, ID>.findByIdOrThrow(
    id: ID, e: Exception = IllegalArgumentException("${T::class.java.name} entity 를 찾을 수 없습니다. id=$id")
): T = findByIdOrNull(id) ?: throw e
```

---


## 일반적으로 id 를 기반으로 엔티티를 조회하는 경우

```kotlin
fun findById(id: Long): Member {
    return memberRepository.findById(id).orElseThrow() // java style
    return memberRepository.findByIdOrNull(id) ?: throw IllegalArgumentException()  // kotlin style
}
```

![image](https://user-images.githubusercontent.com/43930419/224532694-e0b11f0d-e623-478f-8cb2-4af9dadf67e7.png)

- kotlin 에서는 null 인 타입을 구분하므로, 옵셔널을 사용하기보다는 보통 nullable 한 타입으로 나누어서 사용하곤합니다.
- spring data jpa 에서도 이부분을 위해서, `findById` 를 확장시킨 `findByIdOrNull` 를 지원하고 kotlin 에서는 해당 함수를 주로 사용하곤합니다.


```kotlin
fun fun1(id: Long): Member {
    return memberRepository.findByIdOrNull(id) ?: throw IllegalArgumentException()
}

fun fun2(id: Long): Member {
    return memberRepository.findByIdOrNull(id) ?: throw IllegalArgumentException()  // 매번 중복된 처리를 코드마다 해주어야함.
}
```

- 하지만 엔티티를 찾을때마다 null check 를 따로 해주어야한다는 불편함이 남아있습니다.

---

## 확장함수 활용

### 레포지토리에 확장함수를 만들어주기

```kotlin
fun MemberRepository.findByMemberId(id: Long): Member =
    findByIdOrNull(id) ?: throw IllegalArgumentException("member id가 존재하지 않습니다. id: $id")

interface MemberRepository : JpaRepository<Member, Long>
```

```kotlin
fun findById(id: Long): Member {
    return memberRepository.findByMemberId(id)
}
```

- `findByMemberId` 와 같이 확장함수를 이용하게 된다면, 호출하는곳에서 예외처리에 대한 코드를 중복적으로 작성할 필요가 없어지게 됩니다.


```kotlin
fun MemberRepository.findByMemberId(id: Long): Member =
    findByIdOrNull(id) ?: throw IllegalArgumentException("member id가 존재하지 않습니다. id: $id")

fun AEntityRepository.findByAEntityId(id: Long): AEntity =
    findByIdOrNull(id) ?: throw IllegalArgumentException("AEntity id가 존재하지 않습니다. id: $id")

fun MemberRepository.findByBEntityId(id: Long): BEntity =
    findByIdOrNull(id) ?: throw IllegalArgumentException("BEntity id가 존재하지 않습니다. id: $id")
```

- 하지만 이경우 레포지토리 인터페이스마다 따로 만들어줘야하는 불편함은 여전히 존재합니다.


### 공통 확장함수를 만들어주기
> `CrudRepository` 에 대한 확장함수를 만들어줍니다.

```kotlin
inline fun <reified T, ID> CrudRepository<T, ID>.findByIdOrThrow(
    id: ID, e: Exception = IllegalArgumentException("${T::class.java.name} entity 를 찾을 수 없습니다. id=$id")
): T = findByIdOrNull(id) ?: throw e
```

```kotlin
@DisplayName("존재하지 않는 id로 조회하면, 예외가 발생한다2")
@Test
fun findByIdOrThrowTest() {
    shouldThrowExactly<IllegalArgumentException> {
        memberRepository.findByIdOrThrow(100L)
    }
}
```

- `CrudRepository` 를 확장하여 제공하고 있는 kotlin 용 `findByIdOrNull` 처럼 `findByIdOrThrow` 를 만들어줍니다.
- 어떤 엔티티를 조회하다가 에러가 발생했는지를 명시적으로 보기위해 `reified` 를 적용하여 엔티티의 이름을 찾을 수 있게 구성했습니다.

---

## Conclusion
- kotlin 에서 jpa 사용시 확장함수를 이용하여, id 조회의 경우 예외를 내보내는 공통함수를 만들 수 있다.


---

## Code
- 관련된 코드는 [Github](https://github.com/unluckyjung/kotlin-spring-jpa-playground/tree/feat/custom-crud-repository) 에서 볼 수 있습니다.

---

## Reference
- https://kotlinlang.org/docs/extensions.html
- https://codechacha.com/ko/kotlin-reified-keyword/