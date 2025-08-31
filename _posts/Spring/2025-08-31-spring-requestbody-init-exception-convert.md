---
title: RequestBody DTO init 예외가 HttpMessageNotReadableException 으로 래핑되는 문제
date: 2025-08-31-14:30
categories:
- Spring

tags:
- Spring
- DB
- Exception

---

## @RequestBody 바인딩 과정에서 DTO init 사용시 주의할점을 정리해 봅니다.
> 커스텀 예외가 Spring MVC의 메시지 컨버터 단계에서 `HttpMessageNotReadableException` 으로 포장되기 때문에, 원래 의도한 예외 핸들링이 동작하지 않는다.

---

## Goal
- @RequestBody 바인딩 중 DTO 초기화 블록에서 던진 커스텀 예외가 왜 직접 잡히지 않고 HttpMessageNotReadableException으로 래핑되는지 이해한다.
- 실무에서 해당 예외를 올바르게 처리할 수 있는 방법(rootCause 추출)을 정리한다.

---

## 선결론
- Spring MVC는 Jackson에서 발생한 역직렬화 예외를 `HttpMessageNotReadableException` 으로 래핑한다.
- 따라서 핸들러에서 DTO init {…} 의 커스텀 예외를 직접 잡으려면 **exception.rootCause 를 확인해야 한다.**

---

## 이슈
- `@RequestBody` 바인딩 중 DTO init{…} 에서 던진 커스텀 Exception이 이  Jackson → Spring Converter 과정에서 `HttpMessageNotReadableException` 으로 래핑돼 버림.
- 이로인해 커스텀 Exception 을 처리하길 원했던 핸들러가 `@RequestBody` 의 객체 init 안에서 발생한 예외에 대해 의도한대로 동작하지 않음.

## 원인 분석

### Spring MVC 예외 흐름
1. Jackson은 역직렬화 중 발생한 예외를 `JsonMappingException` 으로 감싼다.
2. Spring MVC의 `MessageConverter(AbstractMessageConverterMethodArgumentResolver)`가 이 예외를 다시 `HttpMessageNotReadableException`으로 포장한다.
3. DTO 초기화 `(init {…})` 에서 발생한 커스텀 예외 역시 Jackson → Spring 단계에서 두 번 래핑되므로, 예외가 발생한 root예외는 최종적으로 발생한 rootCause에 위치한다.

## 예외 구조 비교

### DTO init 블록에서 발생한 예외

```
HttpMessageNotReadableException  
 └─ cause: JsonMappingException  
      └─ cause: CustomException
```

### Jackson 역직렬화 단계의 기본 예외

```
HttpMessageNotReadableException  
 └─ cause: InvalidFormatException   (또는 MissingKotlinParameterException)
```

### DTO 예시 코드

```kotlin
data class Dto(
    val name: String,
){
    init {
        require(name.isNotBlank()) { "name must not be blank" }
    }
}
```

---

### 해결 방법
- 핸들러에서 HttpMessageNotReadableException을 그대로 처리하지 말고 **가장 안쪽 원인(rootCause)** 을 확인한다. (참고로 rootCause 는 Spring 에서 wrapping 한 예외타입에만 존재한다.)

```kotlin
@ExceptionHandler(HttpMessageNotReadableException::class)
fun handleHttpMessageNotReadable(ex: HttpMessageNotReadableException): ResponseEntity<Any> {
    val cause = ex.rootCause ?: ex.cause
    return when (cause) {
        is CustomException -> ResponseEntity.badRequest().body(cause.message)
        else -> ResponseEntity.status(HttpStatus.BAD_REQUEST).body("Invalid request")
    }
}
```

---

## Conclusion
- @RequestBody 바인딩시 DTO 내부에서 발생한 예외는 Spring MVC의 Converter 단계에서 HttpMessageNotReadableException 으로 감싸진다.
- 따라서 커스텀 예외를 핸들링하려면 예외 체인의 rootCause를 탐색하는 방식이 필요하다.
- 실무에서는 Jackson 기본 예외(InvalidFormatException, MissingKotlinParameterException)와 DTO init의 커스텀 예외를 구분하여 처리해야 한다.

---

## Reference
- MappingJackson2HttpMessageConverter [(Spring Docs)](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/converter/json/MappingJackson2HttpMessageConverter.html)
- AbstractMessageConverterMethodArgumentResolver [(Spring Docs)](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/mvc/method/annotation/AbstractMessageConverterMethodArgumentResolver.html)

