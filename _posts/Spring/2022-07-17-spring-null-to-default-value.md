---
title: RequestParam, PathVariable 에 디폴트값 넣어주기
date: 2022-07-17-18:00
categories:
- Spring
- Kotlin

tags:
- Spring
- Kotlin


---

## kotlin에서 @RequestParam, @PathVariable 에 디폴트값을 주는 방법을 알아봅니다.
> 주소창에 입력이 없을때 디폴트값을 설정해봅니다.

---

## Goal
- `@RequestParam`, `@PathVariable` 에 디폴트값을 넣어봅니다.
- 코틀린에서 주의할점을 알아봅니다.

---

## @RequestParam
> required 를 false로 받고, value값을 줍니다.

```kotlin
@GetMapping("hello/query-params/default-test1")
fun helloQueryParamsDefaultTest1(
    @RequestParam(defaultValue = "77", required = false) age: Int?,
): String {
    return "age: $age"
}
```

- 코드에서 보는것처럼 `@RequestParam` 옵션을 `required = false` 으로 주고 받는 타입에 `?` 을 붙여서 nullable하게 해줍니다.
- 그리고 `defaultValue = {value}` 값을 넣어주면 됩니다.

### 주의할점

```kotlin
// input null, return null
@GetMapping("hello/query-params/default-test2")
fun helloQueryParamsDefaultTest2(
    @RequestParam(required = false) age: Int? = 77,
): String {
    return "age: $age"
}
```

- 코틀린의 경우 파라메터단에서 디폴트 값을 정해줄 수 있습니다.
- 하지만 `age: Int? = 77` 처럼 값을 주는경우, 언뜻보면 age에 77이라는 값에 들어갈것처럼 보이나 실제로는 `null` 값이 오게 됩니다.
- 이유는 **ArgumentResolver**를 거치며 **age에 null이라는 값이 채워진채**로 받기 때문에, 디폴트값이 삽입되지 않는것입니다. (`RequestParamMethodArgumentResolver`)
- 디폴트값은 아예 인자를 받지 않는 경우에만 작동하는데, 위의 예제에서는 null 값을 받기 때문에 당연하게도 디폴트값이 삽입되지 않기 때문입니다.


## @PathVariable
> required 를 false로 받고, null 인경우에 대한 로직을 직접 구현해줍니다.

```kotlin
@GetMapping("/hello/{age}/default-test1", "/hello/default-test1")
fun pathDefaultTest(
//        @PathVariable(required = false) age: Int? = 10
    @PathVariable(name = "age", required = false) _age: Int?
): String {
    val age = _age ?: 10
    return "age: $age"
}
```
- `@PathVariable` 에 required 옵션을 false로 주는것은 같으나, 해당 어노테이션에서는 디폴트값을 주는 옵션이 존재하지 않습니다.
- 예제 처럼 중간에 pathVariable 이 있는 형태라면, pathVariable을 받지 않는 `"/hello/default-test1"` 와 같은 path 형태를 소화하게 한뒤, nullable 한 형태로 받은뒤 뒤의 로직에서 null일 경우에 대한 로직으로 처리해주면 됩니다.
- `age: Int? = 10` 의 경우에도 `@RequestParam` 에서와 같은 이유로 디폴트값이 삽입 되지 않아 null값으로 받게 됩니다. (`PathVariableMethodArgumentResolver`)


---

## Conclustion
- `required=fasle` 옵션을 통해서 선택적으로 nullable 하게 입력을 받을 수 있다.
- 하지만 `ArgumentResolver` 를 거치고 null이라는 값이 채워져서 들어오기때문에 kotlin 에서의 디폴트값을 주는 방법은 적용되지 않는다.