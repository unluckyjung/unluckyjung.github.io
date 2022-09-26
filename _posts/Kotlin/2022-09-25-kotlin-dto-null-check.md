---
title: Kotlin + Spring 에서 Dto에 null 요청을 받았을때 처리하기
date: 2022-09-25-14:00
categories:
- Kotlin
- Spring

tags:
- Kotlin
- Spring
- Validation

---

## Kotlin 에서 Dto의 NotNull Field에 null 요청을 받았을때 처리하는법을 알아봅니다.
> validation 어노테이션으로 해결하지 못하고, 예외 타입을 찾아 로직으로 해결해야합니다.

---

## Goal
- kotlin + spring 에서 dto 에 **notNull & missType** 요청이 오는경우, 적절하게 처리하는법을 알아봅니다.

---

## 상황

```kotlin
data class Request(
    val name: String,
    val age: Int,
)
```

- Requset DTO가 위와 같은 형태에 정의되어있다고 해보겠습니다.

```json
# Requset

{
  // name 을 요청에 담지 않음. (null)
  "age": "abcd" // "abcd" 를 Int로 변환 불가능.
}
```


- NotNull 형태인 name 에 null 이 담겨서 오거나, age 에 숫자가 아닌 형태의 요청이 오는경우 예외가 발생하게 될것이고, 해당 예외를 적절히 처리해서 클라이언트에게 응답해야합니다.

## 선 결론
> 자바와 다르게 어노테이션이 아닌, 예외 타입을 찾아 로직으로 해결해야합니다.

```kotlin
@ExceptionHandler(value = [HttpMessageNotReadableException::class])
@ResponseBody
fun dtoTypeMissMatchException(exception: HttpMessageNotReadableException): ResponseEntity<ExceptionDto> {
    val msg = when (val causeException = exception.cause) {
        is MissingKotlinParameterException -> {
            "해당 필드는 null 로 오면 안됩니다. 필드명: ${causeException.parameter.name}"
        }

        else -> "요청을 역직렬화 하는과정에서 예외가 발생했습니다."
    }

    return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(
        ExceptionDto(message = msg, value = null)
    )
}
```

- `HttpMessageNotReadableException` 을 처리하는 함수를 만들고, 해당 예외가 터진 `cause` 를 찾아서 분기처리를 통해서 응답해주어야합니다.


---

### kotlin 은 NotNull 타입을 구분한다.
> 비즈니스 로직상, name 은 null이 될 수 없습니다.

```kotlin
data class Request(
    @field: NotNull
    val name: String,
    val age: Int,
)
```

- java 스타일로는 위와 같이 처리할 수 있을것입니다.
- 하지만, 코틀린의 특징상 name 은 `String?` 이 아닌 `String` 으로, 애초에 변수가 null을 허용하지 않는 타입으로 정의 되어있습니다.
- 위와 같은 상황에서 name이 null 로 채워져서 오게 된다면, `@field: NotNull` 어노테이션 을 통한 `validation` 로직을 타기전, **변수에 값을 할당하는 과정에서 예외가 발생** 하게 되어 `@field: NotNull` 은 의미없는 어노테이션이 되게 됩니다.

```kotlin
@field: NotNull
val name: String?
```

- 이를 해결하기 위해서 관련된 글을 찾아보면 위와같이 변수를 Nullable 하게 타입을 바꾸라는 이야기들이 있지만, 해당 방법은 나중에 해당 `Request dto` 를 사용하는 부분에서 비즈니스 로직상 `name` 이 NotNull 임에도 불구하고, null 체크를 해주어야한다는 불편함이 있게 됩니다.

---

## 발생하는 예외 확인 

- NotNull 타입에 Null이 담기려고 할때 발생하는 예외가 무엇인지 확인해보면 `HttpMessageNotReadableException` 이 발생하는 하는것을 확인할 수 있습니다.
- 하지만 정말 예외가 발생한 원인인 `MissingKotlinParameterException` 이랑은 직접적인 상속관계가 아닌것을 확인할 수 있습니다.

> HttpMessageNotReadableException

<img width="382" alt="image" src="https://user-images.githubusercontent.com/43930419/192088958-af852a50-6ea3-4e5a-be82-42eb1f5b7976.png">

> HttpMessageNotReadableException


<img width="346" alt="image" src="https://user-images.githubusercontent.com/43930419/192088966-908fb52c-c025-48fa-a5e8-1ab2ab41fb54.png">


### 원인이 되었던 예외 확인하기

```kotlin
is MissingKotlinParameterException -> {
    val causeException = (exception.cause as MissingKotlinParameterException)
    "Parameter is missing: ${causeException.parameter.name}"
}
```

- 발생한 예외인 `HttpMessageNotReadableException` 의 `cause` 를 확인하면 어떤 타입의 예외가 발생한것이 원인인지를 찾아 해결할 수 있습니다.
- 이후 원인이 되었던 예외 타입으로 형변환을 해준뒤 내부를 확인해보면(when을 이용해 스마트 캐스팅 가능), 실제로 어떤 원인 (`causeException.parameter.name`) 으로 인해 해당 예외가 발생했는지를 명확히 알 수 있게 됩니다. 


---

## 형변환 예외 처리

- Number 형태로 와야하는 요청(`val age: Int`) 에 숫자형태가 아닌 잘못된 요청이 오는 경우도 `HttpMessageNotReadableException` 이 발생하게 됩니다.

```kotlin
is InvalidFormatException -> {
    val causeException = exception.cause as InvalidFormatException
    "입력 받은 ${causeException.value} 를 ${causeException.targetType} 으로 변환중 에러가 발생했습니다."
}
```

- 이 경우에도 마찬가지로 예외가 발생된 원인을 찾아서 처리해주면 됩니다. `(InvalidFormatException)`

---

## Smaple Code
> NotNull 처리, 형 변환 처리


- [Github Code](https://github.com/unluckyjung/kotlin-spring-jpa-playground/tree/spring/dto-type-advice)

```kotlin

@RestControllerAdvice
class GlobalExceptionHandler {

    @ExceptionHandler(value = [HttpMessageNotReadableException::class])
    @ResponseBody
    fun dtoTypeMissMatchException(exception: HttpMessageNotReadableException): ResponseEntity<ExceptionDto> {
        val msg = when (val causeException = exception.cause) {
            is InvalidFormatException -> {
                "입력 받은 ${causeException.value} 를 ${causeException.targetType} 으로 변환중 에러가 발생했습니다."
            }

            is MissingKotlinParameterException -> {
                "Parameter is missing: ${causeException.parameter.name}"
            }

            else -> "요청을 역직렬화 하는과정에서 예외가 발생했습니다."
        }

        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(
            ExceptionDto(message = msg, value = null)
        )
    }
}

data class ExceptionDto(
    val message: String = "default Msg",
    val field: String = "default field",
    val value: Any?,
)
```

---

## Conclusion
- **kotlin + spring** 사용시 NotNull 변수 타입에 null으로 요청이 오는경우, 변수에 null을 먼저 채우려고 시도하게 되어, 자바처럼 어노테이션으로 예외응답을 처리할 수 없습니다.
- `HttpMessageNotReadableException` 예외를 잡아, 어떤 예외가 발생했는지를 확인한후 분기처리를 통해 해결해 주어야 합니다.

