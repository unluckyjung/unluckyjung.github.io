---
title: Spring 캐싱 여부를 테스트 하기
date: 2023-02-11-18:00
categories:
- Spring

tags:
- Spring
- Caching
- Kotlin

---

## Spring 에서 캐싱이 올바르게 되었는지를 테스트 해봅니다.
> 모킹용 도구를 이용합니다.

---

## Goal
- Spring 에서 캐싱이 올바르게 되었는지를 테스트 해봅니다.
- `@SpykBean` 과 같은 모킹용 도구를 통해서 테스트 하는 방법을 알아봅니다.

---

## 실제 비즈니스 로직을 담고 있는 서비스들

```kotlin
@Service
class MemberService(
    private val memberFind: MemberFind,
) {
    fun find(id: Long): Member.Response {
        return memberFind.findById(id = id).run {
            Member.Response(
                id = this.id,
                name = this.name
            )
        }
    }
}

@Transactional(readOnly = true)
@Component
class MemberFind(
    private val memberRepository: MemberRepository,
) {
    fun findById(id: Long): Member {
        return memberRepository.findByIdOrThrow(id = id)
    }
}
```

- `MemberService` 는 `MemberFind` 를 이용해 id에 맞는 멤버가 있는지를 조회하는 구조입니다.
- `MemberService` 에 캐싱을 할수 있도록 설정 해보겠습니다.

> 어노테이션을 통해 스프링 캐시를 진행하는 방법은 [Spring에서 어노테이션을 이용해 cache 적용하기](https://unluckyjung.github.io/spring/kotlin/2022/07/31/spring-cache/) 포스팅을 참고하시면 되겠습니다.

## 캐싱

```kotlin
@EnableCaching
@SpringBootApplication
class KopringApplication

fun main(args: Array<String>) {
    runApplication<KopringApplication>(*args)
}
```

```kotlin
@Service
class MemberCacheService(
    private val memberService: MemberService,
) {
    @Cacheable(value = ["cacheTest"], key = "#id")
    fun findByUseCache(id: Long): Member.Response {
        return memberService.find(id)
    }

    fun findByUseNoCache(id: Long): Member.Response {
        return memberService.find(id)
    }
}
```

- `findByUseCache` 를 호출하는 경우 id 를 기반해 캐싱된 결과를 내보내도록 해주었고
- `findByUseNoCache` 를 호출하는 경우 캐싱된 결과를 내보내지 않도록 하였습니다.

---

## 테스트코드

```kotlin
@IntegrationTest
class MemberCacheServiceTest(
    private val memberCacheService: MemberCacheService,
    private val memberRepository: MemberRepository,
) {
    @SpykBean
    private lateinit var memberService: MemberService

    @DisplayName("캐싱 함수를 통해 호출하는 경우, 같은 아이디로 호출시 캐싱된 결과가 반환된다.")
    @Test
    fun cacheTest() {
        val member = memberRepository.save(Member(name = "unluckyjung"))

        repeat(3) {
            val result = memberCacheService.findByUseCache(member.id)
            result.name shouldBe member.name
        }

        verify(exactly = 1) {
            memberService.find(member.id)
        }
    }

    @DisplayName("캐싱 함수를 통해 호출하지 않는 경우, 같은 아이디로 호출시 캐싱된 결과가 반환되지 않는다.")
    @Test
    fun cacheTest2() {
        val member = memberRepository.save(Member(name = "unluckyjung"))

        repeat(3) {
            val result = memberCacheService.findByUseCache(member.id)
            result.name shouldBe member.name
        }

        verify(exactly = 3) {
            memberService.find(member.id)
        }
    }
}
```

- 3번 호출해본뒤, `verify(exactly = count)` 를 이용하여 `memberService.fun`이 몇번 호출되었는지를 확인하여 테스트해볼 수 있습니다.
- 캐싱된 케이스는 3번을 호출했으나 1번, 캐싱되지 않은 케이스는 3번이 호출되었음을 확인할 수 있습니다.


```gradle
testImplementation("com.ninja-squad:springmockk:3.1.1")
```

```kotlin
@SpykBean
private lateinit var memberService: MemberService
```


- 핵심은 `MemberCacheService` 가 호출하는 `MemberService` 를 모킹한것입니다.
- 해당 케이스는 전체적인 통합테스트의 기준에서의 테스트를 목적으로 삼았고, 동작 자체는 기존의 코드대로 돌아가는것을 의도했기 때문에 [코틀린용 빈 모킹도구](https://github.com/Ninja-Squad/springmockk)인 `@SpykBean` 을 사용했습니다.
- 단순히 캐싱 여부만을 확인하고 싶다면 `MemberService` 함수의 동작까지 모킹한뒤 호출여부를 테스트하셔도 문제는 되지 않습니다.

---

## 생각해볼점

### 참조하고 있는 빈에 의존적인 테스트가 괜찮은가?
- 위와 같은 방법은 캐싱용 서비스를 따로 분리해둔뒤 해당 서비스에서 어떤 서비스를 호출하는지를 알고있어야 테스트가 가능한 방법입니다.
- 만약 캐싱을 `MemberService` 에서 진행하고 `MemberService` 는 다른 빈을 주입받지 않는 상태일때는 위와 같은 방법을 통해 테스트하는것은 불가능합니다.
- 다만 개인적인 생각으로는, 캐싱하는 역할을 가지는 서비스는 따로 분리하는것이 (`MemberCacheService` 레이어를 따로 구분) 역할과 책임 분리성에서도 좋은 구조라고 생각하기 때문에, 해당 방법으로도 테스트해도 충분하다고 생각합니다.
- 빈에 비 의존적인 방법을 통해서 캐싱여부를 확인하는 방법은 차후 필요성이 느껴질때 추가적으로 찾아볼 예정입니다.

### 캐싱 여부를 판단하는것이 괜찮은가?
- 해당 방법은 스프링에서 지원하는 캐싱 기능 자체를 테스트해보는것처럼 느껴질 수 있습니다.
- 마치 예를 들면 이미 구현되어있는 랜덤 함수에서, 진짜 랜덤한 결과가 나오는것인지를 테스트하는 불필요한 테스트를 하는것과 같은 느낌을 받을 수 있습니다.
- 하지만 예시의 경우에는 캐싱의 조건이 아주 간단하지만, 만약 조금더 복잡한 조건과 상황이 들어간 상태에서의 캐싱 유무를 확인해야 한다면 캐싱되었는지를 확인하는 테스트는 필요하다고 생각합니다.
- 테스트를 하는 목적은 여러가지이지만 (코드의 견고성, 리팩토링 등등...), **비즈니스 코드를 작성하는 프로그래머의 마음의 안정** 을 가질수 있게 해주는 목적도 있다고 생각합니다. (저의 경우에도 저를 잘 믿지 않아, 테스트 코드를 현업에서 굉장히 촘촘하게 짜려고 노력하는편입니다.)
- 캐싱 여부자체도 테스트하는것이 필수라고는 말할수 없지만, 캐싱이 필요한경우는 큰 부하를 생각해야하는 상황이기 때문에, 캐싱이 제대로 되지 않았을 경우에 대한 이펙트는 크다고 생각됩니다. (부하테스트 단에서도 캐싱 동작여부를 체크 할 수 있긴하지만, 저희가 QA 분들에게 테스트 전부를 일임하지 않고, E2E 테스트를 작성하는것을 생각해보면 되겠습니다)
- 따라서 해당 방법을 통해서 테스트 코드를 작성하고, 프로그래머가 캐싱이 정상적으로 작동하는것을 확인하고 마음의 안정성을 가질 수 있게된다면, 저는 그 가치만으로도 테스트 코드를 작성하는 의미는 충분하다고 생각합니다.

---

## Conclusion
- Spring 캐싱 동작 여부를 테스트용 모킹 도구를 통해서 테스트 해볼 수 있다.

--- 

## Code
- 관련된 코드는 [Github](https://github.com/unluckyjung/kotlin-spring-jpa-playground/tree/spring/test/caching) 에서 확인하실 수 있습니다.



