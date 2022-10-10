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

## Conclusion

- 캐싱 관련 어노테이션들(`@EnableCaching, @Cacheable, @CachePut, @CacheEvict`)을 이용해, 스프링에서 쉽게 캐싱 등록, 수정, 삭제를 구현할 수 있다.
- 관련된 코드는 [github](https://github.com/unluckyjung/kotlin-spring-jpa-playground/tree/spring/caching-test) 에서 볼 수 있습니다.



---

## Reference
- https://www.baeldung.com/spring-cache-tutorial
- https://stackoverflow.com/questions/8181768/can-i-set-a-ttl-for-cacheable
- https://www.baeldung.com/spring-boot-evict-cache

