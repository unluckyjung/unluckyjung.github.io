---
title: Valid vs Validated (with kotlin)
date: 2022-07-03-18:00
categories:
- Spring

tags:
- Spring
- kotlin
- Annotation


---

## @Valid 와 @Validated 의 사용법과 차이점을 알아봅니다.
> JSR 지원 vs Spring 지원


---

## Goal
- `@Valid` 어노테이션과 `@Validated` 사용법을 알아봅니다.
- `@Valid` 어노테이션과 `@Validated` 의 차이를 알아봅니다.

---

## @Valid
> JSR 지원 스펙

```kotlin
import javax.validation.constraints.NotBlank
import javax.validation.constraints.PositiveOrZero
import javax.validation.constraints.Size

data class MemberTestDto(
    @field:NotBlank
    val name: String,

    @field:PositiveOrZero
    val age: Int,
)
```

```kotlin
@RestController
@RequestMapping("api/v1/member/")
class MemberController(
    val memberService: MemberService
) {
    @GetMapping("valid-get-controller")
    fun getMemberInfoValid(@Valid @RequestBody memberReq: MemberTestDto): String {
      ...
    }
}
```
- Validation관련 어노테이션(ex `@NotBlank`)이 붙은 필드값들을 전부 검사해줍니다.
- 위의 예시에서는 name이 `@NotBlank`인지, age가 `@PositiveOrZero` 인지를 전부 체크해줍니다.
- 컨트롤러 레이어에서 `@RequestBody` 근처에 붙으면 **Argument Resolver** 단에서 처리가 됩니다.
- 이때 `MethodArgumentNotValidException` 이 발생하고, `DefaultHandlerExceptionResolver` 에서 처리되어 **404** 응답으로 처리합니다.


---

## @Validated
> Spring 에서 지원합니다.

- 사용처가 크게 3가지가 있습니다.

1. DTO 에서 일부 필드들만 그룹화 해서 검증하고 싶은 경우
2. 파라메터 형태로 검증 하고 싶은 경우
3. 컨트롤러가 아닌 다른 레이어에서도 `@Valid` 를 통해서 검사를 수행하고 싶은 경우

하나씩 알아보겠습니다.

### 1. DTO 에서 일부 필드들만 그룹화 해서 검증하고 싶은 경우
> 마커인터페이스를 이용해 그룹화 해줍니다.

```kotlin
interface MemberValidationGroup {
    interface AgeCheck
    interface NameCheck
}
```

- 여기서 마커 인터페이스란 이름 가지고 있는 마킹용 인터페이스를 말합니다.
- 위와같이 AgeCheck, NameCheck 인터페이스를 만들어줍니다.

```kotlin
interface NameCheck
```

- 참고: 꼭 인터페이스가 중첩된 상태일필요는 없고, 위처럼 독립적인 인터페이스로 만들어주어도 상관없습니다.

```kotlin
data class MemberTestDto(

    // 그룹 지정
    @field:NotBlank(groups = [MemberValidationGroup.NameCheck::class])
    val name: String,

    // 그룹 지정
    @field:PositiveOrZero(groups = [MemberValidationGroup.AgeCheck::class])
    val age: Int,
)
```

- 속성에 그룹을 정해줍니다.

```kotlin
@GetMapping("validated-get-controller")
fun getMemberInfoValidated(
    // 그룹화 된부분 검사 -> 400 에러
    @Validated(MemberValidationGroup.AgeCheck::class)
    @RequestBody memberReq: MemberTestDto
): String {
    ...
}
```

- 컨트롤러단에서 `@Validated(마커인터페이스)` 를 정해주면 `MemberTestDto` 중 **마커인터페이스에 해당하는 부분** 만 validation이 진행됩니다. 예시의 경우에는 age 부분만 유효성 검증을 진행합니다.
- 이때 역시 **Argument Resolver** 단에서 처리가 되어 **404** 응답을 내게 됩니다.
- 이 방법을 통해 하나의 dto 내에서, 전체 필드가 아닌 일부 필드만 검증할 수 있게 설정할 수 있습니다.


### 1.1 @Valid 와 섞어쓸때 주의해야합니다.  

```kotlin
@GetMapping("validated-get-controller")
fun getMemberInfoValidated(
    // @Valid 사용
    @Valid @RequestBody memberReq: MemberTestDto
): String {
    ...
}

@GetMapping("validated-get-controller")
fun getMemberInfoValidated(
    // 그룹 미지정
    @Validated @RequestBody memberReq: MemberTestDto
): String {
    ...
}
```

1. `@Validated` 에 특정 마커를 정해주지 않는경우
2. field에 그룹이 지정되어있는데 `@Valid` 를 사용하는 경우 
  
- 둘다 **그룹이 정해지지 않은 속성만 검증 작업을 진행합니다.**
- 즉 위와 같이 `@Validted`, `@Valid`를 사용하는 경우, 그룹화가 되어있는 `name, age` 필드는 둘다 **검증 작업이 이루어지지 않게** 됩니다.

```kotlin
data class MemberTestDto(
    @field:NotBlank(groups = [MemberValidationGroup.NameCheck::class])
    val name: String,

    @field:PositiveOrZero(groups = [MemberValidationGroup.AgeCheck::class])
    val age: Int,

    // 그룹화가 안되어있는 속성
    @field:Size(max = 10)
    val nickName: String? = null
)
```

- 만약 `nickName` 처럼 그룹화가 되지 않은 속성이 있다면, 해당 속성만 유효성 검사를 진행하게 됩니다.


### 2. 파라메터 형태로 검증 하고 싶은 경우
> 클래스단에 `@Validated` 를 달아주고, 파라메터에 Validation 관련 어노테이션을 붙여줍니다.

```kotlin
@RestController
@RequestMapping("api/v1/member/")
@Validated
class MemberController {
    ...
    @GetMapping("validated-get-param")
    fun getMemberInfoValidatedParam(
        @Min(value = 20, message = "나이는 20살이상 이여야 합니다.")
        @RequestParam age: Int,
    ): String {
      ...
    }
}
```

- 예시로 클래스단에 `@Validated` 를 달아주고, RequestParam 으로 받는 age에 `@Min` 을 붙이는 경우, `?age=-1` 과 같은 케이스를 잡아줄 수 있습니다.
- 이것이 가능한 이유는 `@Validated` 를 클래스단에 붙여주면, AOP 포인터컷으로 `MethodValidationInterceptor` 가 등록되어 해당 메소드를 수행할때 유효성 검증작업이 이루어지게 됩니다.
- 이때는 Argument Resolver 단에서 처리되는것이 아닌 **MethodValidationInterceptor** 에서 처리되어 **500번 응답**이 발생하고, 다른 예외값인 `ConstraintViolationException` 이 발생하게 됩니다.

> 참고로 컨트롤러 레이어의 클래스단에 `@Validated`를 붙여도, `@Valid` 사용예시는 그대로 **Argument Resolver**에서 처리되어 404 응답을 되돌려줍니다.


### 2.1 혼합해서 사용하는 경우

```kotlin
@GetMapping("validated-get-param2")
fun getMemberInfoValidatedParam2(
    @Valid @RequestBody memberReq: MemberTestDto,
    @Min(value = 20, message = "나이는 20살이상 이여야 합니다.")
    @RequestParam age: Int,
): String {
    ...
}
```

- `@Valid` 도 통과하지 못하고, `@Min` 도 통과하지 못하는 요청이 오는경우, `@Valid` 가 먼저 체크되어 `MethodArgumentValidException`이 발생합니다.
- 만약 `@Valid`는 통과하고, `@Min` 에서 걸리는 경우에는 `ConstraintViolationException` 가 발생합니다.


### 3. 컨트롤러가 아닌 다른 레이어에서도 `@Valid` 를 통해서 검사를 수행하고 싶은 경우
> 클래스에 `@Validated` 를 달아주고, `@Valid` 를 달아줍니다.

- Bean으로 등록되었다면, Controller 레어어가 아닌곳에서도 `@Validated` 을 통해서 유효성 검사가 가능합니다.

```kotlin
@RestController
@RequestMapping("api/v1/member/")
class MemberController(
    val memberService: MemberService
) {
    ...

    @GetMapping("validated-service-layer")
    fun getValidatedService(@RequestBody memberReq: MemberTestDto): String {
        return memberService.validateTest(memberReq)
    }
}


@Validated
@Service
class MemberService {
    // 500 응답
    fun validateTest(@Valid memberReq: MemberTestDto): String {
        ...
    }
}
```

- 예시로 **Service 레이어** 에서 처리하고 싶은경우, 클래스단에 `@Validated` 를 붙여주고 컨트롤러에서 RequestBody를 처리하듯이 파라메터 객체에 `@Valid` 를 붙여주면 됩니다.
- 주의할점은 Controller가 아니므로, Argument Resolver 단에서 처리되는것이 아닌 **MethodValidationInterceptor** 에서 처리되어 **MethodValidationInterceptor** 에서 처리되고 `ConstraintViolationException` 가 발생합니다.

---

## Conclusion
### @Valid
- JSR 지원 스펙이다.
- 컨트롤러 파라메터에서 사용시, 리졸버 단계에서 유효성 검사가 진행된다.
- 발생되는 예외는 `MethodArgumentValidException` 이다.
- **404 응답** 을 내보낸다.


### @Validated
- 스프링에서 지원하는 스펙이다.
- 필드 일부 검사(그룹핑 가능) 가 가능하다.
- 빈으로 등록되어있는 클래스에 붙을수 있고, 이를통해 레이어 구분없이 검증(`@Valid`)기능을 동작하게 할 수 있다.
- 컨트롤러에서, `RequestParam` 이나 `PathVariable`를 검증하고 싶을때 사용할 수 있다.
- AOP를 이용해 인터셉터가 등록되어 처리되어진다.(그룹핑의 용도 제외)
- 발생되는 예외는 `ConstraintViolationException` 이다.
- **500 응답** 을 내보낸다.

---

## Reference
- https://www.baeldung.com/spring-valid-vs-validated
- https://www.baeldung.com/spring-validate-requestparam-pathvariable
- https://www.baeldung.com/javax-validation-groups
- https://mangkyu.tistory.com/174
- https://velog.io/@damiano1027/Spring-Valid-Validated%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EB%8D%B0%EC%9D%B4%ED%84%B0-%EC%9C%A0%ED%9A%A8%EC%84%B1-%EA%B2%80%EC%A6%9D
- https://github.com/cheese10yun/spring-jpa-best-practices/blob/master/doc/step-02.md
- https://velog.io/@freddiey/Spring-validationwith-kotlin