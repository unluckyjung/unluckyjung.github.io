---
title: kotlin mockmvc 사용법
date: 2022-10-16-14:10
categories:
- TestCode
- Kotlin
- MockMvc

tags:
- Kotlin
- TestCode

---

## Spring MockMvc 기본 사용법을 정리합니다.
> `@SpringBootTest` 를 이용해 통합테스트를 기준으로 설명합니다.

---

## Goal
- `mockMvc` 를 이용해 contoller에 http 형식을 요청을 보내고, 응답을 받아봅니다.
- 응답을 받아서 또 다른 요청을 하는법을 알아봅니다.
- 파일을 요청에 담아 보내는법을 알아봅ㄴ니다.

---

## 기본 세팅

## 기본 컨트롤러 세팅

```kotlin
@RequestMapping("test/v1/")
@RestController
class TestController {

    @GetMapping("hello")
    fun hello(): String {
        return "hello"
    }

    @GetMapping("queryParams")
    fun queryParams(@ModelAttribute req: Request): Response {
        return Response(
            req.age,
            req.name
        )
    }

    @ResponseStatus(HttpStatus.CREATED)
    @PostMapping("json-post")
    fun jsonPost(@RequestBody req: Request): Response {
        return Response(
            req.age,
            req.name
        )
    }
}

data class Request(
    val age: Int,
    val name: String,
)

data class Response(
    val age: Int,
    val name: String
)
```

### MockMvc 세팅

```kotlin
@TestConstructor(autowireMode = TestConstructor.AutowireMode.ALL)
@Transactional
@SpringBootTest
@AutoConfigureMockMvc
class MockMvcTest(
    val mockMvc: MockMvc
) {
    ...
}
```

---

## Get
> resonse 200

```kotlin
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get  // import가 이것이 되는지 반드시 확인

@Test
fun helloTest() {
    mockMvc.perform(
//            MockMvcRequestBuilders.get("/test/v1/hello")  // MockMvcRequestBuilders를 써야함 주의
        get("/test/v1/hello")
    ).andExpect(
        status().isOk
    ).andExpect(MockMvcResultMatchers.content().string("hello"))
}
```

- `perform(url)` 을 통해서 http 요청을 보낼 수 있습니다.
- 이후 `andExpect` 를 통해 반환값을 검증할 수 있습니다.

---

## Get
> With QueryParams

```kotlin
@Test
fun queryParamsTest() {

    // 반드시 LinkedMultiValueMap 이여야 함
    val queryParams = LinkedMultiValueMap<String, String>()
    queryParams["age"] = "30"
    queryParams["name"] = "jys"

    mockMvc.perform(
        get("/test/v1/queryParams")
//                .params(queryParams)  // 가능
            .queryParams(queryParams)
    ).andExpect(
        // 하나씩 검증가능
        status().isOk
    ).andExpectAll(
        // 한꺼번에 검증가능
        jsonPath("\$.name").value("jys"),
        jsonPath("\$.age").value("30")
    )
}
```

### import 시 주의
- 기본적으로 `MockMvcRequestBuilders` 을 사용해야합니다.
- `import org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get` 을 직접 import 해주어 prefix를 생략할 수 있습니다.

### 쿼리 파라미터 삽입
- `.queryParams` 혹은 `.params` 을 이용해서 request시 쿼리파라미터를 넣어줄 수 있습니다.
- 이때 사용할 수 있는 Map은 `LinkedMultiValueMap` 입니다. (같은 key값을 가진 여러 value가 오는 경우 list로 받아서 처리하기 위해서)
- 좀더 자세한 내용은 [이글](https://jojoldu.tistory.com/478)을 참고하시면 좋습니다.

### body 값 검증
- 반환값이 body를 가지고 있는 경우 `jsonPath("\$.필드이름").value("값")` 을 통해서 responseBody 를 검증 할 수 있습니다.
- `andExpectAll` 를 이용해 한번에 여러 조건을 체크할 수 있습니다.

---

## Post
> req: body, res: body

```kotlin
@Test
fun jsonPostTest() {

    val request = Request(28, "goodall")

    // object to json
    val jsonBody = jacksonObjectMapper().writeValueAsString(request)

    // ZoeDateTime 사용시
    // val jsonBody = objectMapper.writeValueAsString(request)


    mockMvc.perform(
        post("/test/v1/json-post").content(jsonBody)
            .contentType(MediaType.APPLICATION_JSON)
            .accept(MediaType.APPLICATION_JSON)

    ).andExpectAll(
        jsonPath("\$.name").value("goodall"),
        jsonPath("\$.age").value("28")
    ).andDo(
        // 결과 출력 예시
        MockMvcResultHandlers.print()
    )
}
```

- 일일히 하나씩 검사하기보다는 묶어서 검사하기 원한다면 `andExpectAll` 를 이용하면 됩니다.
- request 에 body를 담고 싶은 경우 `jacksonObjectMapper().writeValueAsString(DTO 형태의 BODY)` 를 넣어서 요청할 수 있습니다.
- `.andDo` 를 이용해 요청결과를 받아볼 수 있습니다.


### ZoneDateTime 사용시 주의할점

```kotlin
val jsonBody = objectMapper.writeValueAsString(request)
// val jsonBody = jacksonObjectMapper().writeValueAsString(request) ZoneDateTime 사용시 불가능
```

- 다만 `ZoneDateTime` 의 경우에는 `jacksonObjectMapper` 가 아닌 `objectMapper.writeValueAsString`  을 사용해야합니다.
- `jacksonObjectMapper` 으로는 `ZoneDateTime` 직렬화가 불가능합니다.

---

## perfom 결과 이용하기

```kotlin
val result = mockMvc.perform(
    MockMvcRequestBuilders.post(url))
        .content(jsonBody)
        .contentType(MediaType.APPLICATION_JSON)
        .accept(MediaType.APPLICATION_JSON)
    ).andReturn()

val response = objectMapper.readValue(result, DTO이름::class.java)

val name = response.name
```

- `.andReturn()` 를 통해 `MvcResult` 를 반환값으로 받을 수 있습니다.
- 이후 `objectMapper.readValue` 를 이용해 원하는 DTO 타입으로 역직렬화 후 반환값을 통해 추가로직을 태울 수 있습니다.

---

## 파일을 업로드 해보기
> MultipartFile 을 테스트 하는경우 사용됩니다.

- 한개의 요청으로 파일과 json 형태의 dto 도 같이 받는것을 예시로 작성했습니다.

### 컨트롤러

```kotlin
@RequestMapping("api/v1/file")
@RestController
class MultiPartController(
    private val fileService: FileService,
) {

    @PostMapping("/{id}")
    fun upload(
        @PathVariable id: Long,
        @RequestPart file: MultipartFile,
        @RequestPart dto: RequestDto,
        @RequestParam filename: String
    ) {
        fileService.upload(id, file, dto, filename)
    }
}

data class RequestDto(
    val name: String
)
```

### 테스트 코드

```kotlin
val csvFile = MockMultipartFile(
    "file",
    "mockFile.csv",
    // String 형태로 넣어줘야 한다.
    "multipart/form-data",
    "1,2,3".toByteArray()
)

/* 같이 전송할 json 파일의 데이터입니다.
{
    name: "jys"
}
*/

val jsonFile = MockMultipartFile("dto", "", "application/json", "{\"name\": \"jys\"}".toByteArray())

val testFilename = "testCodeFile"
val testId = 1L

mockMvc.multipart("/api/v1/file/$testId") {
    file(csvFile) // csv 파일 전송
        .param("filename", testFilename)
        // @RequestParam filename: String 처리를 .part 로도 가능하다.
        // .part(MockPart("filename", testFilename.toByteArray()))
    file(jsonFile)  // json dto 전송
}.andExpect {
    status { isOk() }
}.andDo { }
```

- `mockMvc.perform` 이 아니라 `mockMvc.multipart` 을 사용하는것에 **주의해야합니다.** (이 경우에는 POST 요청으로만 가게됩니다. 다른 메소드로 보내고 싶은 경우에는 [mockMvc 에서 MockMultipartFile 을 put 요청으로 보내기](https://unluckyjung.github.io/testcode/kotlin/mockmultipartfile/spring/mockmvc/2022/10/29/MockMultipartFile-put/) 를 참고하세요.)
- json 형태를 전달하는 경우에는 body 데이터를 `toByteArray()` 을 이용해 변환한뒤 전달해주어야 합니다.

```kotlin
.part(MockPart("파라메터 이름(key)", 파라메터값(value))
.param("filename", testFilename)
```

- `@RequestParam` 를 넣어줄때 위 두가지의 방법 으로 파라메터를 넣어줄 수 있습니다.

---

## Conclusion
- mockmvc 를 이용하여 컨트롤러단 요청에 대한 응답을 테스트할 수 있다.
- 쿼리파라메터 삽압시 `LinkedMultiValueMap` 를 사용해야하는것에 주의해야한다.
- `ZoneDateTime` 을 요청에 넣어주는 경우에는 `objectMapper.writeValueAsString` 을 이용해야한다.
- 파일을 요청에 담아 보내는 경우에는 `MockMultipartFile` 를 이용하고 `mockMvc.multipart` 을 호출해야한다. (기본적으로 POST 요청시)

---

## Code
- https://github.com/unluckyjung/kotlin-spring-jpa-playground/tree/test/mockmvc
- https://github.com/unluckyjung/kotlin-spring-jpa-playground/tree/spring/multipart-with-json