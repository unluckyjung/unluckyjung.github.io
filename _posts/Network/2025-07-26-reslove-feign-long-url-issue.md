---
title: OpenFeign 사용시 긴 URL 로 인한 에러 대응
date: 2025-07-26-15:00
categories:
- Network

tags:
- Network
- Spring
- Kotlin
- Feign
- TroubleShooting

---

## FeignClinet 를 통한 요청시 url 길이에 주의해야한다.
> `@RequestParam`, `pathvariable` 을 통한 요청길이가 너무 길어진경우 4xx 에러를 받을 수 있다.

- FeignClient 를 이용해 원격 API를 호출할 때, @RequestParam이나 @PathVariable로 전달하는 값이 길어지면 HTTP 4xx 에러(특히 414 URI Too Long 또는 400 Bad Request)가 발생할 수 있다.
- 브라우저의 URL 길이 제한과 마찬가지로 서버나 프록시, HTTP 클라이언트에도 헤더 및 요청라인 크기 제한이 있기 때문이다.

---

## 재현

```kotlin
@FeignClient(
    name = "example",
    url = "http://localhost:8080",
    configuration = [FeignConfig::class]
)
interface FeignTestClient {
    @GetMapping("/example/{tx}")
    fun submit(@PathVariable tx: String): String
}


// 호출 예시
val longTx = "a".repeat(3000)
client.submit(longTx) // → 414 또는 400 에러
```

- 위와 같은 코드가 있을때 `tx` 의 길이가 아주 긴 길이(3000이상) 으로 들어오는경우 에러가 발생할 수 있다.
- 이유는 springboot3 기준 [공식문서](https://docs.spring.io/spring-boot/appendix/application-properties/index.html#application-properties.server.server.max-http-request-header-size) 를 보면 길이가 `8KB` 이라는것을 확인할 수 있다.

### Spring Boot 3 기본 한계: max-http-request-header-size
- 프로퍼티: server.max-http-request-header-size
- 디폴트: 8 KB (8 192바이트)
- 적용 대상: Tomcat, Jetty, Undertow 공통

> 이 값을 넘어서면 톰캣 기준 org.apache.coyote.http11.AbstractHttp11Processor에서 Response.sendError(400) 처리

---

## 해결방안

### 헤더 크기 제한 늘리기

```yml
# application.yml 예시
server:
  # Spring Boot 2.1+ 또는 3.x 기준
  max-http-request-header-size: 20KB
```

- 20 KB 정도로 여유 있게 설정하면 3,000자 ~ 5,000자 수준의 URL도 소화 가능
- 클라우드 환경(프록시, AWS ELB 등)을 쓴다면, 앞단 로드밸런서 설정도 함께 늘려야 함

### GET 요청을 POST 요청을 바꿔 처리한다.
> 긴 파라미터는 URL이 아니라 요청 본문(request body) 에 담도록 API 설계 변경

- **팀 내에서는 이 방법으로 해결함**


```kotlin
@FeignClient("example", url = "http://localhost:8080")
interface FeignTestClient {
    @PostMapping("/example")
    fun submit(@RequestBody payload: TxRequest): String
}

data class TxRequest(val tx: String)
```

---

## Conclusion
- Feign을 통한 원격 호출에서 긴 URL로 인한 4xx 에러를 만났다면,
- 헤더 크기 설정(server.max-http-request-header-size)을 늘리거나
- GET → POST 전환으로 본문 전송 을 고려해보자.


---

## Reference
- https://docs.spring.io/spring-boot/appendix/application-properties/index.html#application-properties.server.server.max-http-request-header-size
- https://github.com/OpenFeign/feign/issues/865
- https://docs.spring.io/spring-cloud-openfeign/docs/current/reference/html/appendix.html
- https://www.baeldung.com/spring-boot-max-http-header-size
