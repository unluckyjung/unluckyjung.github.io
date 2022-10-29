---
title: mockMvc 에서 MockMultipartFile 을 put 요청으로 보내기
date: 2022-10-29-18:10
categories:
- TestCode
- Kotlin
- MockMultipartFile
- Spring
- MockMvc

tags:
- Kotlin
- TestCode

---

## mockMvc 에서 MockMultipartFile 을 put 요청으로 보내는 법을 알아봅니다.
> `MockMvcRequestBuilders` 를 이용합니다.

---

## Goal
- `MockMvc` 를 이용해 `MockMultipartFile` 을 **PUT** METHOD 로 보내는법을 알아봅니다.

---

## MockMvcRequestBuilders 를 사용하는방법
> 요청시 메소드를 직접 설정해줄 수 있습니다.

```kotlin
val file = MockMultipartFile(
    "file",
    "mockFile.csv",
    "multipart/form-data",
    "1,2,3".toByteArray()
)

val jsonBody = MockMultipartFile(
    "body",
    "",
    "application/json",
    "{\"age\": \""30\", \"name\": \"jys\"}".toByteArray()
)

//  POST 요청으로만 가게 되어짐.
//  mockMvc.multipart("url") {
//      file(file)
//      file(jsonBody)
//  }.andExpect {
//      status { isOk() }
//  }.andDo { }

val builder = MockMvcRequestBuilders.multipart("url")
builder.with { request ->
    request.method = "PUT"
    request
}
mockMvc.perform(builder.file(file).file(jsonBody))
    .andExpectAll(
        MockMvcResultMatchers.status().isOk,
    )
```

- 위의 예시는 file 업로드와 jsonBody 를 한개의 요청으로 보내는 예시입니다.
- 기본적으로 사용하는 `mockMvc.multipart` 을 이용해서 컨트롤러에 요청을 보내는 방식은 항상 `POST` 로만 요청을 보내게 됩니다.

```kotlin
val builder = MockMvcRequestBuilders.multipart("url")
builder.with { request ->
    request.method = "PUT"
    request
}
```

- 이를 해결하기 위해서는, 위와 같이 빌더를 새롭게 만들어 준뒤 요청으로 보낼 HTTP METHOD 를 명시 해준뒤, `mockMvc.perform` 메서드안의 파라메터로 넣어주면 됩니다.

---

## Conclustion
- `MockMvcRequestBuilders` 를 이용하여 요청 보낼 HTTP METHOD 를 명시해줄수 있다.
- 이 방법을 이용하여 `mockMvc.multipart` 가 아닌 `mockMvc.perform(빌더)` 를 이용하여 파일 업로드시 PUT 으로 요청을 보낼 수 있다.

---

## Reference
- https://stackoverflow.com/questions/38571716/how-to-put-multipart-form-data-using-spring-mockmvc
- https://stackoverflow.com/questions/62862635/mockmvc-calling-a-put-endpoint-that-accepts-a-multipart-file
