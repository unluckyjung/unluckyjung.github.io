---
title: Spring kotlin RedisTemplate 사용법
date: 2023-03-11-15:00
categories:
- Spring

tags:
- Spring
- Kotlin
- Database

---

## RedisTemplate (Spring Redis) 사용법

---

## Goal
- Kopring 상황에서의 redisTemplate 기본세팅 및 사용법을 정리해봅니다.

---

## 기본 설정

### Infra 설정

```yml
# docker-compose.yml

version: '3.7'
services:
    redis:
      container_name: redis
      hostname: redis6379
      image: redis:latest
      command: redis-server
      labels:
        - "name=redis"
        - "mode=standalone"
      ports:
        - 6379:6379
```

- redis 를 로컬에 띄우기위한 기본적인 `docker-compose` 입니다.

```console
// redis 다운
$ docker pull redis

// docker-compose 구동
$ docker-compose -d

// redis 콘솔 접속
$ docker exec -it redis redis-cli
```


### Spring 설정

```gradle
implementation("org.springframework.boot:spring-boot-starter-data-redis")
```

```yml
spring:
  redis:
    host: localhost # container host-name 이랑은 별개이다.
    port: 6379 # default port
```

```kotlin
@Configuration
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
    fun redisTemplate(): RedisTemplate<*, *> {
        return RedisTemplate<Any, Any>().apply {
            this.setConnectionFactory(redisConnectionFactory())

            // "\xac\xed\x00" 같은 불필요한 해시값을 보지 않기 위해 serializer 설정
            this.keySerializer = StringRedisSerializer()
            this.hashKeySerializer = StringRedisSerializer()
            this.valueSerializer = StringRedisSerializer()
        }
    }
}
```

- `RedisConnectionFactory` 빈을 먼저 만들어줍니다.
- 이후 `RedisConnectionFactory` 를 사용하는 `redisTemplate` 빈을 만들어주어 기본적인 RedisTemplate 를 사용할 수 있습니다.
- `StringRedisSerializer()` 는 **serializer** 설정을 해주는것인데, 생략할시 redis 콘솔에서 출력했을때 `"\xac\xed\x00"` 같은 값이 prefix 에 붙어 노출되게 됩니다. 관련 [docs](https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/#redis.streams.receive.serialization)

> `StringRedisSerializer` 를 생략한 경우

![image](https://user-images.githubusercontent.com/43930419/214362875-881aba1a-9d86-44f8-98ca-b0cc5d358791.png)

---

## value가 String 형태의 저장/조회

### 만들어준 redisTemplate 사용

```kotlin
class RedisConfigTest(
    private val redisTemplate: RedisTemplate<String, Any>,
) {
    @DisplayName("string 기반의 key")
    @Test
    fun stringCacheTest() {
        val valueOperations = redisTemplate.opsForValue()
        val key = "stringKey"
        val value = "goodall"

        // 삽입
        valueOperations[key] = "goodall"

        // 조회
        valueOperations[key] shouldBe value
    }
}
```

- `RedisConfig` 에서 만들어준 `redisTemplate` 를 주입받아서 사용할 수 있습니다.
- `opsForValue()` 를 이용해서 key, value 를 넣어줄 수 있는 `ValueOperations` 를 획득할 수 있습니다.
- 조회의 경우에는 `get` or `[]` 를 이용해서 할 수 있습니다.


```console
$ 127.0.0.1:6379> get stringKey
"goodall"
```

### String 특화 StringRedisTemplate 사용

```kotlin
private val stringRedisTemplate: StringRedisTemplate,

@DisplayName("stringRedisTemplate 기반의 key")
@Test
fun stringRedisObjectTest() {
    val valueOperations = stringRedisTemplate.opsForValue()
    val key = "stringKey"
    val value = "goodall"

    // 삽입
    valueOperations[key] = "goodall"

    // 조회
    valueOperations[key] shouldBe value
}
```

```console
$ 127.0.0.1:6379> get stringKey
"goodall"
```

- config 에서 설정/주입한 RedisTemplate 빈 말고도, 기본적으로 제공하는 `StringRedisTemplate` 를 이용할 수 도 있습니다.

---

## Set

```kotlin
@DisplayName("set 기반의 key")
@Test
fun setCacheTest() {
    val valueOperations = redisTemplate.opsForSet()
    val key = "setKey"

    // 삽입
    valueOperations.add(key, "unluckyjung", "goodall")

    // 조회
    valueOperations.members(key)?.shouldContain("unluckyjung")
    valueOperations.members(key)?.size shouldBe 2
}
```

- set 의 경우 `opsForSet()` 를 호출해서 Set 을 다를 수 있는 `valueOperations` 를 얻을 수 있습니다.
- key 하나에 여러값을 담을 수 있습니다. 
- 조회시에는 `members(key)` 를 통해서 보유하고 있는 values 를 획들 할 수 있습니다.

```console
$ 127.0.0.1:6379> smembers setKey
1) "goodall"
2) "unluckyjung"
```

---

## Hash

```kotlin
@Bean
fun hashRedisTemplate(): RedisTemplate<*, *> {
    return RedisTemplate<Any, Any>().apply {
        this.setConnectionFactory(redisConnectionFactory())

        // "\xac\xed\x00" 같은 불필요한 해시값을 보지 않기 위해 serializer 설정
        this.keySerializer = StringRedisSerializer()
        this.hashKeySerializer = StringRedisSerializer()
        this.hashValueSerializer = StringRedisSerializer()
    }
}
```

- 해시의 경우 해시 해시에 대한 key, value 도 **serializer** 설정을 해주어야 해시값을 제외한 값을 명확히 볼 수 있습니다.

```kotlin
@DisplayName("hash 기반의 key")
@Test
fun hashCacheTest() {
    val hashOperations = hashRedisTemplate.opsForHash<String, String>()

    val key = "hashKey"
    val hashKey = "goodall"
    val value = "yoonsung"
    hashOperations.put(key, hashKey, value)

    hashOperations.get(key, hashKey) shouldBe value

    val entries = hashOperations.entries(key)

    entries[hashKey] shouldBe value
}
```

- `opsForHash` 을 이용해주면됩니다. 이때, `<HashKeyType, HashValueType>` 을 넣어줄 수 있습니다.
- 삽입의 경우에는 `hashOperations.put(key, hashKey, value)` 형태로 사용합니다.
- 조회시에는 `hashOperations.entries(key)` 를 통해서 key 에 해당하는 해시키를 찾은뒤, `entries[hashKey]` 를 통해서 value 에 접근할 수 있습니다.

```console
$ 127.0.0.1:6379> hkeys hashKey
1) "goodall"

$ 127.0.0.1:6379> hget hashKey goodall
"yoonsung"
```

---

## 만료시간 설정

```kotlin
@DisplayName("key 에 따른 3초후 삭제 옵션을 주면, 3초후 조회되지 않는다.")
@Test
fun stringRedisDeleteWaitTest() {
    val valueOperations = stringRedisTemplate.opsForValue()
    val key = "expireKey"
    val value = "goodall"

    valueOperations[key] = value

    // 3초뒤 삭제
    redisTemplate.expire(key, 3, TimeUnit.SECONDS)
    valueOperations[key] shouldBe value

    Thread.sleep(3500)
    valueOperations[key] shouldBe null
}
```

- `redisTemplate.expire(key, 3, TimeUnit.SECONDS)` 처럼 key 에 따른 만료시간을 설정해줄 수 있습니다.


---

## Value 가 Object 형태일때


### 설정

```kotlin
data class RedisObject(
    val name: String,
    val age: Int,
) : Serializable
```

- 저장되는것이 Object 인 형태인 경우에는, 직렬화가 가능하도록 `Serializable` 을 반드시 붙여주어야합니다.

```kotlin
@Bean
fun objectRedisTemplate(): RedisTemplate<*, *> {
    return RedisTemplate<Any, Any>().apply {
        this.setConnectionFactory(redisConnectionFactory())
        this.keySerializer = StringRedisSerializer()

        // 직렬화 / 역직렬화시 json(dto) 형태의 값 형태로 저장하기 위한 설정
        this.valueSerializer = Jackson2JsonRedisSerializer(RedisObject::class.java).also {
            // 디폴트 생성자 처리를위해 명시적으로 코틀린 모듈 ObjectMapper 삽입
            it.setObjectMapper(jacksonObjectMapper())
        }
    }
}
```

- Object 저장을 할 수 있는 RedisTemplate 를 하나 새롭게 구성해줍니다.

```kotlin
// 직렬화 / 역직렬화시 json(dto) 형태의 값 형태로 저장하기 위한 설정
this.valueSerializer = Jackson2JsonRedisSerializer(RedisObject::class.java).also {
    // 디폴트 생성자 처리를위해 명시적으로 코틀린 모듈 ObjectMapper 삽입
    it.setObjectMapper(jacksonObjectMapper())
}
```

- 중요한것은 `jascksonObjectMapper` 을 명시적으로 설정 해주어야합니다.
- 해당 설정이 없으면, 조회시 `deserialize` 과정에서 기본생성자가 없다는 에러가 발생하게 됩니다.


> org.springframework.data.redis.serializer.SerializationException: Could not read JSON: Cannot construct instance of ...

> org.springframework.data.redis.serializer.SerializationException: (no Creators, like default construct, exist): cannot deserialize from Object value (no delegate- or property-based Creator)

```kotlin
data class RedisObject(
    val name: String = "",
    val age: Int = 0,
) : Serializable
```

- 또 다른 방법으로는 디폴트값을 명시적으로 넣어준다면, 자동적으로 디폴트 생성자가 생성되어 위와같은 에러가 발생하지 않게 됩니다.

> 기본적으로 SpringBoot 설정에 들어가있는 `jackson-module-kotlin` 이 존재한다면, spring jackson 에서 사용하는 리플랙션을 위해 자동으로 디폴트 생성자 처리를 해주게되는데, redisTemplate 설정에서는 자동으로 설정이 되지 않아서 이런문제가 발생하게 됩니다. 관련된 글은 [이곳](https://unluckyjung.github.io/kotlin/spring/2022/09/10/kotlin-pretendtoknow/#%EC%9E%AD%EC%8A%A8-%EC%BD%94%ED%8B%80%EB%A6%B0-%EB%AA%A8%EB%93%88)을 참고하시면 되시겠습니다.


### 삽입, 조회

```kotlin
private val objectRedisTemplate: RedisTemplate<String, RedisObject>,

@DisplayName("key: string, value: ObjectTest")
@Test
fun stringCacheObjectTest() {
    val valueOperations: ValueOperations<String, RedisObject> = objectRedisTemplate.opsForValue()
    val key = "string-object-key"
    val objectValue = RedisObject("yoonsung", age = 30)

    valueOperations[key] = objectValue

    valueOperations[key] shouldBe objectValue
}
```

- 설정 부분만 제대로 해주었다면, String 다루듯이 동일하게 처리하면됩니다.

```kotlin
$ 127.0.0.1:6379> get string-object-key
"{\"name\":\"yoonsung\",\"age\":30}"
```

---

## Conclusion
- 세팅시에 hash 와 관련된 값들은 `StringRedisSerializer()` 를 명시적으로 넣어주어야한다.
- String 사용시 `stringRedisTemplate` 를 용할 수 있다.
- Object 형태를 직렬화하여 저장할시, `Jackson2JsonRedisSerializer` 와 `setObjectMapper(jacksonObjectMapper())` 를 명시적으로 넣어주어야 한다.

---

## Code
- 관련된 코드는 [Github](https://github.com/unluckyjung/kotlin-spring-jpa-playground/tree/spring/redis-template) 에서 볼 수 있습니다.


---

## Reference
- https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/
- https://redis.io/commands/
- https://bcp0109.tistory.com/328
- https://dev-monkey-dugi.tistory.com/150