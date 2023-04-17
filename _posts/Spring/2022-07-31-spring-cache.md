---
title: Spring에서 어노테이션을 이용해 cache 적용하기
date: 2022-07-31-18:00
categories:
- Spring
- Kotlin

tags:
- Spring
- Cache


---

## 스프링에서 캐싱기능을 사용해봅니다.
> 어노테이션을 이용한 캐싱 기능을 사용해봅니다.

---

## Goal
- `@EnableCaching` + `@Cacheable` 를 이용해 캐시를 적용하는법을 알아봅니다.
- `@CachePut` 을 이용해 캐시를 업데이트 해봅니다.
- `@CacheEvit` 을 이용해 캐시를 삭제하는 방법을 알아봅니다.
- 스케쥴링 옵션을 넣어서 작동으로 삭제도 해봅니다.

---

## 캐싱 설정
> `@EnableCaching`을 이용해 캐싱기능 활성화, 함수에 `@Cacheable` 을 이용해 캐싱 타겟 설정


```kotlin
@EnableCaching
class KopringApplication


@Configuration
@EnableCaching
class CacheConfig {}
```

- `@EnableCaching` 를 어플리케이션 or 캐싱설정에 붙여줍니다.
- **ChacheManager** 를 직접 다루는 방법은 이글에서 설명하지 않습니다. (추후 다른글에서 설명할 수도 있습니다.)


```kotlin
@Service
class DummyService {

    //    @Cacheable(value = ["cacheTest"], key = "#name")
    @Cacheable(cacheNames = ["cacheTest"])
    fun cachedFun(name: String): Response {
        return Response(name)
    }

    @Cacheable(value = ["cacheTest2"], key = "#name + #age")
    fun cachedFun2(name: String, age: Int, nickName: String): Response {
        return Response(name)
    }

    @Cacheable(value = ["cacheTest3"], key = "#req.name")
    fun cachedFun3(req: Request): Response {
        return Response(req.name)
    }
}

class Response(
    val name: String
)

class Request(
    val name: String
)
```

> 캐싱 관련 어노테이션은 AOP + Proxy 기반 으로 작동하기 때문에, **내부 호출(Self invocation)을 하는경우에는 작동하지 않습니다.**

- 어느 캐시에 저장할것인지 (`value` or `cacheNames`), 어떤것을 기준으로 캐싱할것인지 (`key`)를 정해 줄 수 있습니다.
- key를 명시해주지 않으면, 파라메터의 모든 값이 key가 됩니다.
- 여러개의 파라메터를 하나의 key 값으로 묶고 싶은경우에는  `+` 를 이용해 주면 됩니다.
- 객체 형식의 파라메터를 받는 경우에도 `#req.name` 과 같은 형식으로 객체내 필드값으로 키값을 정해줄 수 있습니다.


```kotlin
@CacheConfig(cacheNames = ["myCache"])
@Service
class DummyService {}
```

- 클래스내 모든 메서드에서 같은 캐시테이블을 사용할 예정이면 `@CacheConfig` 를 이용해 전역적으로 옵션을 줄수도 있습니다.

## 캐시 업데이트
> `@CachePut` 을 이용합니다.


```kotlin
@CachePut(value = ["cacheTest"])
fun cachePut1(name: String) {
}

@CachePut(value = ["cacheTest"], condition = "#name == 'jys'")
fun cachePut2(name: String) {
}
```

- 캐시에 들어가있는 정보를 업데이트 하고 싶은 경우가 있을 수 있습니다.
- 이경우에는 `@CachePut` 을 이용하고, value에 업데이트 하고 싶은 캐시테이블 이름을 넣어주면 됩니다.
- 또한, **condition** 과 같은 옵션을 넣어서, 캐싱에 필요한 조건식을 같이 넣어줄 수 있습니다.


### 객체를 기반으로 한 캐싱

```kotlin
class Request(
    val name: String,
    val age: Int,
) 

@Cacheable(cacheNames = ["cacheTest"], key = "#req")
fun cachedObjectFun2(req: Request): Response {
    Counter.countUp()
    return Response(req.name)
}
```

- 객체를 그대로 캐싱하는 경우, 해시코드 값을 기반으로 캐싱을 진행하게 됩니다.

```kotlin
@DisplayName("새로운 객체이면, spel 기반이라 같은 값을 주어도 캐싱된 결과가 반환되지 않는다.")
@Test
fun cacheObjectTest3() {
    // data class 인경우, hashCode 가 같아서 캐싱가능 (예시의 Request 는 일반 class)
    val req = Request("goodall", 31)
    val cachedValue = dummyService.cachedObjectFun2(req)

    for (i in 1..3) {
        val newReq = Request("goodall", 31)
        cachedValue shouldNotBe dummyService.cachedObjectFun2(newReq)
    }
    Counter.count shouldBe 4
}
```

- 따라서 데이터를 동일하게 담고 있어도, 구현이 되어있지 않다면, 캐싱된 결과를 반환하지 않게 됩니다.
- 객체 기반으로 캐싱을 원하는 경우, `equals()`, `hashCode()` 를 Request 에 구현 해주거나, kotlin 의 경우에는 data class 를 사용하시면 되겠습니다.

### 파라메터가 없는 경우를 캐싱하는방법

```kotlin
@Cacheable(value = ["cacheTest8"], key = "#root.method.name")
fun cacheNoParam() {
}
```

- 종종 key 값이 마땅한것이 없지만, 캐싱한 결과를 내보내고 싶을 수 있습니다.
- 이 경우에는 `key = "#root.method.name"` or `#root.methodName` 처럼 메서드 이름을 key 값으로 잡고, 처리해주면 됩니다.

---

## 캐싱 조건

### condition
> 특정 조건일 경우 캐싱

```kotlin
class Request(
    val name: String,
    val age: Int,
    val type: CacheType = CacheType.NON,
) {
    fun isNeedCache(): Boolean {
        return type == CacheType.YES && name == "goodall"
    }
}

@Cacheable(value = ["cacheTest5"], key = "#req.name", condition = "#req.name != 'goodall'")
fun cachedWithCondition(req: Request): Response {
    Counter.countUp()
    return Response(req.name)
}

@Cacheable(value = ["cacheTest6"], key = "#req.name + #req.type", condition = "#req.isNeedCache() == true")
fun cachedWithConditionFun(req: Request): Response {
    Counter.countUp()
    return Response(req.name)
}
```

- `condition` 을 통해서 캐싱 조건을 줄 수 있습니다.
- 함수를 이용해서도 조건절 처리가 가능합니다.

### unless
> 캐싱되지 않을 조건

```kotlin
@Cacheable(value = ["cacheTest7"], key = "#req.name", unless = "#req.name.length() > 10")
fun cachedWithUnless(req: Request): Response {
    // #req.isNeedCache().not() 와 같이 kotlin 에서만 제공하는 함수(not())는 사용불가 
    // req.type == CacheType.NON 와 같이 Enum 을 주는 경우도 사용불가.

    Counter.countUp()
    return Response(req.name)
}
```

- 특정 조건의 경우, 캐싱을 원하지 않는다면 `unless` 를 이용해 구분이 가능합니다.
- 주의할점은 조건으로 `Enum` 을 주는 경우, `SpEL` 을 통해 해석하지못해 에러가 발생하게 됩니다.
- 해당 경우에는 `req.isNeedCache()` 와 같이 조건문으로 풀어서 해결할 수 있습니다.
- 또한 `SpEL` 은 기본적으로 자바 문법을 기반으로 해석할 수 있으므로, kotlin 에서만 제공하는 문법은 사용할 수 없습니다. 
- 다만 어느정도는 지원이 되는지 `req.name.length()` => `req.name.length` 정도는 해석하고 있습니다. 이부분은 `SpEL` 을 좀더 파보아야겠습니다. `2023-02-08`

---


## 캐시 삭제
> `@CacheEvict` 을 이용합니다.


```kotlin
@CacheEvict(value = ["cacheTest"])
fun eraseCache(name: String) {
}
```

- 캐싱 기능인 `@Cacheable` 와 완전히 똑같이 옵션을 주면 됩니다.
- 해당 메서드가 호출되는 경우, `name` 에 적용되어있던 캐싱 정보가 삭제됩니다.


### 캐싱 주기적 삭제
> `@Scheduled` 를 이용합니다.


```kotlin
@CacheEvict(cacheNames = ["cacheTest3"], allEntries = true)
@Scheduled(fixedDelay = 1 * 1000)  // 1초마다 호출
fun eraseCache() {
}
```

- `Evict` 을 주기적으로 호출할 수 있도록 `@Scheduled` 를 이용해주면됩니다.
- `@Scheduled` 가 붙은 메서드는 파라메터를 가지면 안되고, public 해야한다는것에 주의해야합니다.
- 예제의 경우 `allEntries = true` 옵션을 통해 `cacheTest3` 캐시에 담겨있던 정보를 1초 간격으로 전부 삭제하게 됩니다.


---

## 번외
- [캐싱된 기능을 테스트 하는 방법](https://unluckyjung.github.io/spring/2023/02/11/spring-cache-test/)

---

## Conclusion

- 캐싱 관련 어노테이션들(`@EnableCaching, @Cacheable, @CachePut, @CacheEvict`)을 이용해, 스프링에서 쉽게 캐싱 등록, 수정, 삭제를 구현할 수 있다.
- 관련된 코드는 [github](https://github.com/unluckyjung/kotlin-spring-jpa-playground/tree/spring/caching-test) 에서 볼 수 있습니다.



---

## Reference
- https://docs.spring.io/spring-framework/docs/3.2.x/spring-framework-reference/html/cache.html
- https://www.baeldung.com/spring-cache-tutorial
- https://stackoverflow.com/questions/8181768/can-i-set-a-ttl-for-cacheable
- https://www.baeldung.com/spring-boot-evict-cache
- https://stackoverflow.com/questions/48888760/cachable-on-methods-without-input-parameters

