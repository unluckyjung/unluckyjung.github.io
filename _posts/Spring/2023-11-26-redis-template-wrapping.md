---
title: Spring RedisTemplate wrapping for object save and get
date: 2023-11-26-20:30
categories:
- Spring

tags:
- Spring
- DB
- Redis
- RedisTemplate

---

## Spring RedisTemplate 사용시, 객체 형태도 저장, 조회가 쉽도록 구현해봅니다.
> Object 를 Json 형태로 변환한뒤, String 으로 변환시켜 저장, 조회 합니다.

---

## Goal
- `RedisTemplate` 를 좀더 쉽게 다룰수 있도록 `RedisTemplate` 를 Wrapping 한 클래스를 만들어봅니다.
- Object 의 경우에도 삽입, 조회가 가능하도록 기능을 구현해봅니다.

---

## Config
> Serializer 를 전부 String 으로 처리해줍니다.

```kotlin
import org.springframework.beans.factory.annotation.Value
import org.springframework.cache.annotation.EnableCaching
import org.springframework.context.annotation.Bean
import org.springframework.context.annotation.Configuration
import org.springframework.data.redis.connection.RedisConnectionFactory
import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory
import org.springframework.data.redis.core.RedisTemplate
import org.springframework.data.redis.serializer.StringRedisSerializer

@Configuration
@EnableCaching
class RedisConfig(
    @Value("\${spring.redis.host}")
    val host: String,

    @Value("\${spring.redis.port}")
    val port: Int,
) {

    @Bean
    fun redisConnectionFactory(): RedisConnectionFactory {
        return LettuceConnectionFactory(host, port)
    }

    @Bean
    fun redisTemplate(): RedisTemplate<String, Any> {
        return RedisTemplate<String, Any>().apply {
            this.setConnectionFactory(redisConnectionFactory())

            this.keySerializer = StringRedisSerializer()
            this.valueSerializer = StringRedisSerializer()

            this.hashKeySerializer = StringRedisSerializer()
            this.hashValueSerializer = StringRedisSerializer()
        }
    }
}
```

- `Serializer` 를 전부 String 으로 통일해 주었습니다.

### StringRedisSerializer 으로 전부 다루는 이유
> String 으로 바꾸어 다루는것이 가장 유연하다고 판단 (개인적인 의견)

- Redis 에 Object 혹은 여러 데이터를 넣기 위해 직렬화/비직렬화 하는 방법에는 여러가지 방법이 있습니다.
- 하지만 Object 별로 RedisTemplate 를 매번 따로 만들어주어야 하거나, pakage 경로가 들어가거나, 직렬화 관련 인터페이스를 상속해야하는등 하나씩 이슈가 있습니다.
- 이에 대한 방안으로 아예 Object 를 Json 형태로 변경하여 String 으로 만들어 Redis 에서 in/out 하는 방법을 선택 했습니다.

---

## 삽입, 조회 코드

```kotlin
import com.fasterxml.jackson.databind.ObjectMapper
import org.springframework.data.redis.core.RedisTemplate
import org.springframework.stereotype.Component
import java.util.concurrent.TimeUnit

@Component
class RedisKeyValueStore(
    private val redisTemplate: RedisTemplate<String, Any>,
    private val objectMapper: ObjectMapper,
) {

    fun save(key: String, value: Any, timeOut: RedisKeyValueTimeOut? = null) {

        timeOut?.let {
            redisTemplate.opsForValue().set(
                key,
                objectMapper.writeValueAsString(value),
                timeOut.time,
                timeOut.timeUnit
            )
        } ?: run {
            redisTemplate.opsForValue().set(
                key,
                objectMapper.writeValueAsString(value),
            )
        }
    }

    fun delete(key: String){
        redisTemplate.delete(key)
    }

    fun <T> getByKey(key: String, clazz: Class<T>): T? {
        val result = redisTemplate.opsForValue()[key].toString()
        return if (result.isEmpty()) null
        else {
            return objectMapper.readValue(result, clazz)
        }
    }

    operator fun <T> get(key: String, clazz: Class<T>): T? {
        return getByKey(key = key, clazz = clazz)
    }

    operator fun set(key: String, value: Any) {
        return save(key = key, value = value)
    }
}

class RedisKeyValueTimeOut(
    val time: Long,
    val timeUnit: TimeUnit,
)
```

- 주의깊게 봐야할 부분은 `getByKey` 입니다.
- 조회시 원하는 타입으로 변환이 필요하기 때문에 clazz 타입을 파라메터로 넣어서 변환된 객체를 반환하도록 하였습니다.
- 또한 get, set operator 를 구현하여 `[]` 대괄호로 저장, 조회가 가능하도록 구현하였습니다.


```kotlin
data class UnluckyJung(
    val name: String,
    val age: Int, // 새롭게 추가된 경우
)

@IntegrationTest
internal class RedisKeyValueStoreTest(
    private val redisKeyValueStore: RedisKeyValueStore,
    private val redisTemplate: RedisTemplate<String, Any>,
) {
    @BeforeEach
    internal fun setUp() {
        flushRedis()
    }

    @AfterEach
    internal fun tearDown() {
        redisTemplate.connectionFactory?.connection?.flushAll()
    }

    private fun flushRedis() {
        redisTemplate.connectionFactory?.connection?.flushAll()
    }


    @DisplayName("object 를 String 으로 변환해서 저장, 조회할 수 있다.")
    @Test
    fun saveTest() {
        val key = "yoonsung"
        val value = UnluckyJung(name = "goodall", age = 30)
        redisKeyValueStore[key] = value

        val result = redisKeyValueStore[key, UnluckyJung::class.java]
        result shouldBe value
    }
}
```


---

## 주의할점
> Object 에 필드가 추가 되고 동기화가 안된 경우, 새롭게 추가된 필드는 조회시 null 로 처리됩니다.

```kotlin
data class UnluckyJung(
    val name: String,
    val age: Int, // 새롭게 추가된 경우
)
```

- UnluckyJung 객체가 최초 name 만 있는상태에서 Redis 에 저장되었다고 가정해보겠습니다.
- 이후 notnull type 인 age 가 추가된 후 조회하게 된다면, age 는 notnull 타입이지만, null 로 차있을 수 있다는점을 고려해야합니다.
- 이유는 objectMapper 처리과정이 결국 reflection 을 사용하기 떄문입니다.
- 좀더 자세한 내용은 [Kotlin 의 변수가 NotNull type 이지만 null 이 들어갈수 있다.](https://unluckyjung.github.io/kotlin/2023/01/27/kotlin-notnull-but-null-case/) 포스팅을 참고하시는게 좋으시겠습니다.

---

## Conclusion
- Spring 에서 `RedisTemplate` 를 다룰때 직렬화를 전부 String 형태로 다루게 할 수 있다.
- 위 방법을 이용하면 Object 를 저장하는 과정에서, Redis Config 를 통일할 수 있고 보일러 플레이트를 줄일 수 있다.
- 다만 리플렉션을 이용하는 부분이 있어, notnull Type 에 Null 이 삽입될 수 있다.

---

## Code
- 해당 코드는 전부 [Github](https://github.com/unluckyjung/kotlin-spring-jpa-playground/pull/11) 에서 보실 수 있습니다.

---