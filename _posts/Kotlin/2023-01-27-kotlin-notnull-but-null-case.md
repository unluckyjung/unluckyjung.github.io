---
title: Kotlin 의 변수가 NotNull type 이지만 null 이 들어갈수 있다.
date: 2023-01-27-00:00
categories:
- Kotlin

tags:
- Kotlin

---

## Kotlin 의 변수가 NotNull 타입이지만, Null 이 저장될 수 있는 케이스를 알아봅니다.
> spring, jpa 를 사용할때 접할 수 있는 케이스를 알아 봅니다.

---

## Goal
- Kotlin 사용시 변수가 NotNull 타입이지만, null 로 저장될 수 있는 케이스를 정리해봅니다.
- 왜 null 로 값이 할당될 수 있는지 알아 봅니다.

---

## 테스트 환경 버전
- spring boot: `2.6.7`
- spring data jpa: `2.6.4`

## Kotlin 의 null 처리

```kotlin
val name: String? // nullable
val name: String // notNull
```

- 코틀린을 사용하시는 분들은 기본적으로 코틀린은 Null 타입 구분을 `?` 을 통해 언어적으로 분리하고 있는것을 알고 계실겁니다.
- 하지만, `String` 와 같이 NotNull 인 경우에도 null 이 저장될 수 있습니다.


---

## [예시1] Spring MVC
> `Collection<NotNull Type>` 형태의 요청에서 null 이 삽입될 수 있습니다.

### Dto 형태

```kotlin
data class TestDto(
    val list: List<Long>, // <== NotNull Type Long
)
```

- `TestDto` 에서 들고 있는 list 는 `List<Long>` 으로, `Long?` 이 아닌 NotNull 타입 으로 선언 해주었습니다.

### 로직

```kotlin
@RequestMapping("api/v1/error")
@RestController
class ErrorTestController {

    @GetMapping("/list-empty-string")
    fun listEmptyStringNullTest(@RequestBody dto: TestDto) {
        dto.list.forEach {
            log.info("number: $it")
        }
    }

    companion object {
        val log = LoggerFactory.getLogger(this::class.java)
    }
}
```

- dto 에서 list 를 받아서 출력하는 간단한 코드입니다.

---

### 일반적인 에러가 나는 요청

```console
GET http://localhost:8080/api/v1/error/list-empty-string
Content-Type: application/json

{
  "list": [20,21,"yoonsung"] // Long 으로 받아야 하지만, "yoonsung" 으로 number 가 아닌형태로 요청
}
```

- [20,21,"yoonsung"] 는 매핑 과정에서  

> JSON parse error: Cannot deserialize value of type `long` from String` 

에러가나서, 애초에 로직을 타지 않는것을 확인할 수 있습니다.


### 문제가 되는 Request

```console
POST http://localhost:8080/api/v1/post/test1
Content-Type: application/json

{
  "list": ["20","21",""]
}

// or

{
  "list": ["20","21",null]
}
```

![image](https://user-images.githubusercontent.com/43930419/212965068-fbf60afc-16c9-485e-9cd8-e8e061da9466.png)

![image](https://user-images.githubusercontent.com/43930419/212965289-f8d6f5ef-1201-42e0-a8ce-cc4373efd45b.png)

- 20, 21 까지 처리 하다가 `""` or `null` 를 처리하는 시점에 가서야 `"java.lang.NullPointerException"` 이 발생하게 됩니다.
- 즉 dto 에 매핑과정에서 오류가 발생하는것이 아닌, **실제 null 로 차있는 element 에 접근하는 시점에 가서야 NPE 가 발생** 한다는것에 주의해야 합니다.


![image](https://user-images.githubusercontent.com/43930419/212964305-afc00a48-3ad7-4834-b0e0-28a6ce39259d.png)

- 디버그 모드로 보았을때, 3번째 element 인 Long 이 NotNull type 이지만 null 로 차있는것을 확인할 수 있습니다.

## 원인(추측)
> reflection

- 해당 이슈가 발생하는 원인은 `,` 단위로 데이터를 자른뒤 필드에 매핑하게 되는데요. 이때 `spring boot` 에서 기본적으로 사용하는 `Jackson` 은 기본적으로 **reflection** 을 통해 역직렬화를 하게 됩니다. 
- 이 과정에서 premitive 타입을 따로 구분하지 않는 코틀린의 경우, `Long` 인 필드에 null 이 채워지는 상황이 발생할 수 있게 되는것으로 예측하고 있습니다. (reflection 을 이용하면, final 한 변수도 변경할 수 있게 됩니다. [관련 내용](https://unluckyjung.github.io/java/2022/02/04/java-reflection-change-final-value/))

- 실제로 Kotlin NotNull Type 의 경우, java 로 디컴파일을 해보는 경우 `@NotNull` 어노테이션을 이용한 형태로 존재하는것을 확인할 수 있습니다. ([디컴파일 하는방법](https://unluckyjung.github.io/kotlin/intellij/2022/06/06/kotlin-decompile/))

```kotlin
class Goodall(val name: String)
```

- 위와 같은 kotlin 코드도 디컴파일을 해보면

![image](https://user-images.githubusercontent.com/43930419/214829372-c5719168-ea57-4c23-b658-c7ad52547b1d.png)

- `@NotNull` 어노테이션에 의존적인 형태인것을 볼수 있습니다.
- 즉, `@NotNull` 을 통한 null validition 이 되기전에 null 이 삽입된것으로 보입니다.

> `2023-01.26` 작성. 내부코드를 열어보고, kotlin + spring 에서의 동작과정을 정리한뒤 수정 예정

---


### `List<String>`
> `Long` 일때랑 동일한 문제가 발생 합니다.

```kotlin
// Request
{
  "list": ["jys","goodall", null]
}

data class TestDto(
    val list: List<String>,
)
```

![image](https://user-images.githubusercontent.com/43930419/212967560-0b5a91d5-e3f1-4800-999b-bd22608e12ca.png)


- 3번쨰 element 가 null 인것을 확인할 수 있습니다.

---

## [예시2] Spring Data JPA

## DB 데이터
<img width="817" alt="image" src="https://user-images.githubusercontent.com/43930419/212958492-92a3fa79-a022-4b55-a7d9-f002014bb099.png">

<img width="296" alt="image" src="https://user-images.githubusercontent.com/43930419/212958040-fa880b7e-a862-463d-8f8e-6ec616c602b2.png">

- id, name 을 컬럼으로 가진 테이블이 있습니다.
- 이때 name 은 nullable 하고, 실제로 데이터는 null 이 차있도록 해보겠습니다. `{id:2, name: null}`

### Entity 구조

```kotlin
@Table(name = "members")
@Entity
class Member(
    @Column(name = "name")
    val name: String, // <=== not null

    @Column(name = "id")
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long = 0L
)
```

- JPA 에서 사용할 엔티티를 만듭니다.
- 이때 name 은 notNull 인 `String` 형태로 만들어주었습니다.

## 조회

```kotlin
@Transactional
fun contextLoads() {
    memberRepository.findAll().forEach { member ->
        log.info("name: ${member.name}")
    }
}

companion object {
    val log = LoggerFactory.getLogger(this::class.java)
}
```

![image](https://user-images.githubusercontent.com/43930419/212960996-70142273-0a49-4b7d-a3af-4832874cee8e.png)

![image](https://user-images.githubusercontent.com/43930419/212961318-55285577-938d-4d6f-a8d4-e915952a61a2.png)

- 엔티티를 조회하고, `name` 필드를 보면 사진과 같이 null 이 할당되어있는것을 볼 수 있습니다.
- `val String` 이 notNull 타입이라고 생각했지만, 해당 필드에 접근했을때 null 이 나와서 NPE 가 발생할 수 있습니다.
- 즉 **DB 내의 데이터가 nullable 하고 실제로 null 데이터**가 들어가있다면, kotlin 의 NotNull Field 에 null 이 들어갈 수 있게됩니다.
- 실제 운영중인 DB의 스키마와, JPA 엔티티에서 사용하는 필드들간의 notNull 조건 유무가 같은지 확인하는것을 권장드립니다.

> JPA 의 경우에도 reflection 을 통해서 데이터를 삽입하게 되는데, mvc 에서 발생한 이유와 원인이 같다고 추측중입니다.


---

## Conclusion
- Kotlin 에서 NotNull 이라고해서, 완벽하게 NotNull 이 보장되지는 않는다.
- 리플랙션(추측)이 사용되는 라이브러리, 프레임워크의 경우에는 NotNull Type 임에도 null 이 채워질 수 있음을 고려하고 있자.

---

## Reference
- https://discuss.kotlinlang.org/t/null-safety-and-java-reflection/8525/17

