---
title: kotlin querydsl proejction
date: 2023-10-09-21:00
categories:
- Querydsl

tags:
- Querydsl
- Spring
- Kotlin

---

## 코틀린에서 querydsl 의 프로젝션을 하는방법을 정리합니다.
> 여러방법이 있지만 `@QueryProjection` 사용을 추천합니다.

---

## Goal
- 코틀린에서 querydsl 의 projection 을 사용하는 방법을 정리해봅니다.
- 방법별 특징을 알아보고, 어떤것을 사용할지 고민해 봅니다.

---

## 어노테이션을 통한 프로젝션 (추천)
> 컴파일타임에 에러를 잡을 수 있습니다.

```kotlin
// QueryDSL 에 의존적인 Dto가 된다.
// 생성자 위에 달아주어야 하기 때문에, 코틀린의 경우 constructor 를 명시해주어야한다.
data class MemberDtoByAnnotation @QueryProjection constructor(
    val name: String,
    val age: Int,
)
```

- 어노테이션을 통한 프로젝션 방법입니다.
- 생성자 위에 달아주어야하므로 kotlin 의 경우,  constructor 을 선언하고 `@QueryProjection` 을 붙여줍니다.
- 이후 Qclass 생성을 위해 **컴파일을 한번 해줍니다.**
  
```kotlin
@DisplayName("[추천] QueryPojection 어노테이션을 이용하면, 잘못된 위치에 다른 타입의 값을 넣는경우 컴파일 타임에 예외가 잡힌다.")
@Test
fun byAnnotation() {
    memberRepository.save(
        Member(
            name = "yoonsung",
            age = 30,
        )
    )

    val result = queryFactory.select(
        QMemberDtoByAnnotation(
            QMember.member.name,
            QMember.member.age,
        )
    ).from(QMember.member).fetchOne()!!

    result.name shouldBe "yoonsung"
    result.age shouldBe 31
}
```

- 이후 조회쪽에서 select 절에 Qclass 를 이용하여 반환 해주면됩니다.
- 이방법의 경우 컴파일을 한번 해주어야하는 귀찮음이 존재하지만, argument 에 대한 타입 체크가 들어가기 때문에 name(string), age(Int) 의 순서를 잘못 넣는경우 **컴파일 타임에 에러가 발생하게 됩니다.**
- 즉, 에러를 사전에 찾을수 있다는 장점이 크게 작용합니다.


---


## 생성자를 통한 프로젝션
> 편리하지만, 컴파일타임때 에러를 잡지 못함.

```kotlin
data class MemberDtoByConstructor(
    val name: String,
    val age: Int,
)
```

- 생성자를 이용한 프로젝션을 위해 data class 를 만들어 줍니다.
- 이떄 보면 `@QueryProjection` 사용하지 않은것을 볼 수 있습니다.

```kotlin
@DisplayName("[보통] Projections.constructor 을 이용하면, 필드를 이용해서 값을 채운다. 다만 필드 위치가 정확히 맞아야한다.")
@Test
fun byConstructor() {
    memberRepository.save(
        Member(
            name = "yoonsung",
            age = 30,
        )
    )

    val result = queryFactory.select(
        Projections.constructor(
            MemberDtoByConstructor::class.java,
            QMember.member.name,
            QMember.member.age,
        )
    ).from(QMember.member).fetchOne()!!

    result.name shouldBe "yoonsung"
    result.age shouldBe 30

    // 잘못된 위치의 경우 예외발생
    shouldThrowExactly<ExpressionException> {
        queryFactory.select(
            Projections.constructor(
                MemberDtoByConstructor::class.java,
                QMember.member.age, // 잘못된 위치
                QMember.member.name,
            )
        ).from(QMember.member).fetchOne()!!
    }
}
```

```kotlin
Projections.constructor(
    Dto이름::class.java,
    Q필드 이름1,
    Q필드 이름2,
)
```

- 프로젝션을 사용할떄 위와 같은 방식으로 값을 채워주면됩니다.
- querydsl 에 비의존적이고 편리한 방법이나, 필드 이름에 맞지 않는 타입을 넣는경우 런타임때 가서야 예외가 발생하게 됩니다.
- 위의 예시 코드를 보면 `MemberDtoByConstructor` 의 경우에는, `name(String), age(Int)` 순으로 선언되어 있습니다.
- 이때 프로젝션을 하는과정에서, `age(Int), name(String)` 과 같이 순서를 바꿔 넣는경우 해당 메서드가 호출되는 타이밍에 가서야 예외가 발생하게 됩니다.
- 저의 경우에는 이런 이유 때문에 해당 방법보다는 `@QueryProjection` 을 이용한 조회방법을 선호합니다.


> 이 밖에도 , 리플랙션을 이용한 `Projections.fields`, setter 를 이용한 `Projections.bean` 을 사용해 프로젝션을 하는 방법이 있으나 그다지 추천하지 않아 정리하지 않겠습니다.

---

## Conclusion
- QueryDsl 에서 원하는 필드만 골라내어 반환하는 프로젝션을 하는 방법이 여러개가 있다.
- 그중에서는 `@QueryProjection` 이 querydsl 에 의존적인 dto 를 만들게 되지만, 컴파일타임때 에러를 잡을수 있다는 큰 장점이 있어 추천한다.