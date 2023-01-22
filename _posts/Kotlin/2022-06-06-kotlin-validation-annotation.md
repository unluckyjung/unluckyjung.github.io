---
title: Kotlin 에서 Validation 사용시 주의할 점
date: 2022-06-06-22:00
categories:
- Kotlin
- Spring

tags:
- Kotlin
- Spring
- Validation

---

## Kotlin 에서 Validation 사용시 주의할 점을 알아봅니다.
> `use-site-target`을 이용해 어노테이션이 붙는 위치를 정해주어야 합니다.

---

## Goal
- Kotlin 에서 **Validation** 사용시 주의할점을 알아봅니다.
- `use-site-target` 을 사용해 어노테이션이 달리는 위치를 명시적으로 정해주어 문제를 해결해봅니다.

---

## 주의할 점
> `use-site-target` 를 이용해 필드에 `Validation` 관련 어노테이션이 달리게 해주어야 합니다.

```kotlin
class Member(
    @field: NotBlank
    val name: String
)
```


---

## 문제가 되는 상황

```kotlin
import com.example.kopring.BaseEntity
import java.time.ZonedDateTime
import javax.persistence.*
import javax.validation.constraints.NotBlank

@Entity
class Member(

    ...

) : BaseEntity(createdAt = createdAt) {
    data class Request(
        @NotBlank(message = "이름은 공백으로 이루어져있을 수 없습니다.")
        val name: String
    )
}
```

- `Member.Request`의 **name** 은 `""` 혹은 `"  "` 와 같은 공백이 되지않게 하는것이 니즈입니다.
- 이를 위해서 `@NotBlank` 어노테이션을 **name 프로퍼티** 에 달아 봅시다.

### 테스트코드

```kotlin
class RequestTest {
    private val validator = Validation.buildDefaultValidatorFactory().validator

    @DisplayName("이름이 공백이면 Validation 의 validator 에 예외가 쌓인다.")
    @Test
    fun nameBlankFailTest() {

        val memberReq = Member.Request(" ")
        val violations = validator.validate(memberReq)

        violations.size shouldBe 1

        for (failMsg in violations) {
            failMsg.message shouldMatch ("이름은 공백으로 이루어져있을 수 없습니다.")
        }
    }
}
```

> **참고:** Validation 어노테이션을 사용하는 경우, **유효성 검증 작업은 필드에 값이 들어간다고 바로 수행되지 않습니다.**  
> 스프링에서 `@Valid` 혹은 `@Validated` 를 이용해 검증을 특정 메서드가 호출시에 유효성 검사를 진행하게 하는것이 일반적이나, 스프링에 의존하지 않는 테스트를 만들기 위해서 `buildDefaultValidatorFactory()` 를 사용했습니다.  
> **이부분에 대한 내용은 이 글에서 자세히 다루지 않고 나중에 다른 글에서 설명할 예정입니다.** `22.06.06`


![image](https://user-images.githubusercontent.com/43930419/172169897-0f7904a3-f8be-4e87-8fb5-fac9091ed42d.png)


- `Member.Request(" ")` 이므로 유효성 검사에서 필터링 되어야 하는 시나리오입니다.
- 하지만 **Validation** 이 제대로 수행되었나 테스트를 해보면, 유효성 검증 단계에서 `" "` 의 경우를 잡지 못한것을 확인할 수 있습니다.

---

## Validation이 안되는 원인
> 필드 변수에 `@NotBlank` 가 붙지 않았습니다.

```kotlin
class Member(
    @NotBlank
    val name: String
)
```

- 위와 같은 코틀린 코드를 작성했을때, 디컴파일을 진행해보면 ([디컴파일 방법을 다룬 포스팅](https://unluckyjung.github.io/kotlin/intellij/2022/06/06/kotlin-decompile/)) 아래의 자바 코드가 나오는 것을 확인할 수 있습니다.

```java
public final class Member {
   @NotNull
   private final String name;

   @NotNull
   public final String getName() {
      return this.name;
   }

   public Member(@NotBlank @NotNull String name) {
      Intrinsics.checkNotNullParameter(name, "name");
      super();
      this.name = name;
   }
}
```

- 여기서 집중해서 보아야 하는점은 `@NotBlank` 어노테이션이 생성자에만 붙어있고, **필드 변수인 `name` 에는 붙어있지 않다는것입니다.**


### 생성자에만 valid 어노테이션이 붙는 원인
> 문제가 발생하는 원인은 해당 **어노테이션이 사용되는 위치가 생성자** 이기 때문입니다.

- 저희는 코틀린을 사용할때, 일반적으로 프로퍼티를 위해서 아래 예시중 Case1 으로 처리하게 됩니다.

```kotlin
// Case1 (Validation 미수행)
class Member(
    @NotBlank <- 생성자 위치에 프로퍼티 선언과 같이 달아둠
    val name: String
)

// Case2 (Validtion 수행)
class Request(
   name: String // 생성자에서 변수 선언안함.
) {
    @NotBlank <- 생성자쪽이 아닌 필드 프로퍼티에 어노테이션을 달아둠
    val name: String = name
}
```

- Case 1 과 같은 상태에서 Validation 관련 어노테이션을 달아주는경우 코틀린에서 어노테이션이 달리는 동작순서는 아래와 같습니다.

```
1. param - (a constructor parameter)
2. property - (the Kotlin's property, it is not accessible from Java code)
3. field - (field)
```

- **Case1** 의 경우 생성자 위치에서 어노테션을 달아주었기 때문에, 1번에서 끊겨 `use-site target` 을 설정해주지 않는경우 target 에 constructor 가 명시되있는것을 찾고, 생성자쪽 파라메터에만 어노테이션이 붙게되어 **필드 프로퍼티쪽에는 어노테이션이 붙지 않게 됩니다.**
- **Case2** 와 같은 경우에는 생성자 위치가아닌, 필드에 어노테이션을 달아 주었기 때문에 정상적으로 필드 프로퍼티쪽에 어노테이션이 적용되는것을 바이트코드를 열어보면 확인할 수 있습니다.
- 하지만 굳이 필드 프로퍼티로 해둘 필요가 없기때문에 명시적으로 `use-site target` 을 이용해서 생성자 프로퍼티를 사용하며 어노테이션을 달아주는 방법을 선택합니다.


---

## 해결방법
> `@field:` 를 이용해서 해당 어노테이션이 붙어야할 위치를 명시적으로 정해주면 됩니다.

```kotlin
// AS-IS
data class Request(
    @NotBlank(message = "이름은 공백으로 이루어져있을 수 없습니다.")
    val name: String
)

// TO-BE
data class Request(
    // @field: 추가
    @field: NotBlank(message = "이름은 공백으로 이루어져있을 수 없습니다.")
    val name: String
)
```

- 만약 생성자 파라메터에도 붙기를 희망한다면, `@param:` 도 같이 붙여주면 됩니다.
- 좀더 다양한 `use-site-target` 의 종류는 [공식문서](https://kotlinlang.org/docs/annotations.html#annotation-use-site-targets) 를 참고하시는것이 도움이 되겠습니다.


---

## Conclusion
- kotlin에서 `Validation`을 사용하는 경우, `@field:` 와 같은 `use-site-target` 를 이용해서 필드에 검증용 어노테이션을 달아준다.
- `Validation` 외에도 자바기반의 어노테이션 라이브러리를 사용하는경우, 해당 어노테이션이 어디에 붙어야 있어야하는지를 확인해볼 필요성이 있다.

---

## Code
- 관련된 코드는 [Github](https://github.com/unluckyjung/kotlin-spring-jpa-playground/blob/main/kopring/src/main/kotlin/com/example/kopring/member/domain/Member.kt) 에서 보실 수 있습니다.

---

## Reference
- https://kotlinlang.org/docs/annotations.html#annotation-use-site-targets
- https://www.baeldung.com/kotlin/annotations
