---
title: JPA 사용시 엔티티의 일부 필드만 조회 해오기
date: 2023-03-01-18:00
categories:
- JPA

tags:
- JPA
- Kotlin

---

## JPA 사용시 엔티티의 컬럼중 일부 필드만 조회 해오는법을 알아봅니다.
> 매핑용 인터페이스를 만든뒤, 해당 인터페이스를 반환하게 합니다.

---


## Goal
- 엔티티 전체를 조회해오는것이 아닌 컬럼중 일부 필드만 조회 해오는법을 알아봅니다.
- QueryDSL 사용없이 JPA 만 사용했을때 가져오는법을 알아봅니다.

---

## 엔티티 구조

```kotlin
@Table(name = "member2")
@Entity
class Member2(
    @Column(name = "name")
    val name: String,

    @Column(name = "nick_name")
    val nickName: String,

    @Column(name = "id", nullable = false)
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long = 0L
)
```

- `Member2` 엔티티티는 id, name, nickName 3개의 필드를 가지고 있습니다.
- 이때 `Member2` 엔티티 전체를 조회해 오는것이 아닌, name 을 제외하여 조회 해오고 싶은 니즈가 있다고 가정 해보겠습니다.

---

## 매핑용 인터페이스 구현 및 조회 코드 작성
> 원하는 필드만 가진 인터페이스 구현, 조회시 앞에서 만든 인터페이스를 반환

```kotlin
interface Member2ProjectionData {
    val id: Long
    val nickName: String
}
```

- 원하는 필드만 가진 매핑용 **인터페이스**를 만들어줍니다. 

```kotlin
interface Member2Repository : JpaRepository<Member2, Long> {
    fun findByName(name: String): Member2ProjectionData? // <=  Member2 엔티티를 반환하지않고, 매핑용 인터페이스를 반환
}
```

- 반환타입을 매핑용 인터페이스로 해줍니다.

![image](https://user-images.githubusercontent.com/43930419/220897439-e26cd38e-f210-4bf3-8fca-2f47b82ca0e9.png)

- 조회시 발생하는 쿼리를 보면, name 은 제외하고 조회쿼리가 나가는것을 볼 수 있습니다.

---

## Conclusion
- 엔티티중 일부 필드(컬럼)만 골라서 조회해오고 싶은경우에는 매핑용 **인터페이스**를 만들어 조회할 수 있다.
- 여러 엔티티를 조합하여, 반환하고 싶은경우에는 `queryDSL` 의 **Projection** 을 사용해야한다.
