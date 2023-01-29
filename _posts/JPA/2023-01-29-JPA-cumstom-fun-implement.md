---
title: JPA Repository 에 커스텀 메서드(함수) 만들고 사용하기
date: 2023-01-29-18:00
categories:
- JPA

tags:
- JPA

---

## JPA 사용시 커스텀 하게 만든 함수를 기본 Repository 에서 사용하게 하기
> 인터페이스를 적절하게 이용하여 구현해줍니다.

---


## Goal
- 인터페이스이기 때문에 구현체가 없는 JPA `Repository` 에 커스텀한 기능을 확장해서 구현하는법을 알아봅니다.

---

## Repository 에서 사용할 커스텀 메서드(함수) 구현
> 인터페이스를 구현해주고, 해당 인터페이스를 상속한 구현체를 만들어줍니다.

```kotlin
// [1] 새로운 인터페이스를 만들고 추가할 기능 정의
interface CustomMemberRepository {
    fun getMemberNameWithPrefix(member: Member): String
}

// [2] 실제 로직을 가진 구현체 구현 (주의: 반드시 상속할 인터페이스의 이름과 동일하게 하고 뒤에 Impl 을 붙여줌)
class CustomMemberRepositoryImpl(
    // JPA 에서 지원하지 않는 jdbcTemplate 를 사용한 로직. EX) BatchInsert
    private val jdbcTemplate: JdbcTemplate,
) : CustomMemberRepository {
    override fun getMemberNameWithPrefix(member: Member): String {
        return "prefix: ${member.name}"
    }
}
```

- 주의할점은 `CustomMemberRepository` 을 상속한 실제 로직을 들고 있는 클래스는, 기본적으로 인테퍼에스의 이름명 뒤에 **Impl** 가 붙은 `XXRepositoryImpl()` 라는 클래스 이름을 가지게 작성해야합니다.

> 참고: 예시의 코드의 경우에는 완전히 적절한 예시가 아니지만, 종종 JPA 에서 지원하지 않는 기능을 `jdbcTemplate` 를 통해서 구현해주어야 하는일들이 종종 있습니다 **(ex Batch Insert)**, 
> 이런 경우 Repository 를 또하나 따로 구현 하는것보다는, 위와 같은 방법을 통해서 레포지토리는 하나로 두고, 메서드(함수) 만 커스텀 된것을 사용하게 구현해줄 수 있습니다.

### 사용

```kotlin
// 기본 인터페이스 Repisotry(MemberRepository) 에서 CustomMemberRepository 를 implement 해줍니다.
interface MemberRepository : JpaRepository<Member, Long>, CustomMemberRepository {
    fun findByName(name: String): Member?
    fun existsByName(name: String): Boolean
}
```

- 실제 로직에서 사용할 레포지토리 인터페이스(`MemberRepository`) 에서는 **[1] 기능 정의 인터페이스** 를 상속해야 합니다. 실제로 구현체가 만들어져 있는 **[2] 를 상속하지 않아야함** 에 주의해야 합니다.

```kotlin
@RepositoryTest
class MemberRepositoryTest(
    private val memberRepository: MemberRepository
) {
    @DisplayName("custom fun 을 호출할 수 있다.")
    @Test
    fun customRepositoryFunTest() {
        val name = "unluckyjung"
        val member = Member(name = name)
        memberRepository.getMemberNameWithPrefix(member) shouldBe "prefix: $name"
    }
}
```

- `memberRepository.getMemberNameWithPrefix(member)` 와 같은 형태로 custom fun 을 호출 할 수 있게됩니다.

---

## Conclusion
- [공식문서](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.custom-implementations)에서 가이드한 방법을 통해, 인터페이스 상속을 적절히 하여 커스텀하게 만든 함수를 인터페이스 형태인 기존 Repository 에서 사용할 수 있다.
- 주의할점은 기본적으로 구현체에는 suffix 에 **Impl** 라는 붙여주어야하고, 로직에서 접근할 인터페이스는 실제구현체가 담겨있는 클래스가 아닌, 상위 인터페이스를 상속 해주어야한다.

---


## Reference
- https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.custom-implementations
