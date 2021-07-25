---
title: Spring 에서 Url을 파싱하기
date: 2021-07-25-17:00
categories:
- Spring

tags:
- Spring
- AntPathMatcher

---

## Spring 에서 Url내부의 데이터를 파싱해 봅니다.
> 정규식을 사용하지 않고 `AntPathMatcher` 를 이용해 파싱해봅니다.

---

## Goal
- 문자열 형태의 URL 에서 원하는 이용해 데이터를 파싱해봅니다.
- 직접 정규식을 짜지 않고, 스프링에서 제공하는 api를 이용해 데이터를 파싱해봅니다.
- `AntPathMatcher` 를 사용해봅니다.

## 개요
> url에서 원하는 정보를 뽑아야 하는 상황이 발생

web 개발을 하다보면, url에서 원하는 데이터를 파싱해야되는 경우가 잦게 발생한다.  
이 문제 사황을 해결하기위해 스프링에서는 많은 기능을 제공해주고 있다.  

`@PathVariable`의 경우가 그중 하나의 예시로 볼 수 있겠다.  

```java
@RestController
@RequestMapping("/uri-pattern")
public class UriPatternController {

    @GetMapping("/users/{id}")
    public ResponseEntity<User> pathVariable(@PathVariable Long id) {
        User user = new User(id, "이름", "email");
        return ResponseEntity.ok().body(user);
    }
}
```

```java
// url : /uri-pattern/uers/77

@GetMapping("/users/{id}")
pathVariable(@PathVariable Long id)
```

이번에는 컨트롤러 단이 아닌 인터셉터 단에서, URL에서 원하는 정보만 파싱해야 되는 상황이 발생했다.  

```java
"/topic/rooms/1/chat"

// 아래와 같이 roomId에만 관심을 가지는 상황
"/topic/rooms/{roomId}/chat" 
```

- 처음에 시도해본 방법은 `roomdId`를 파싱하는 정규식을 작성하는것이었다.
- 하지만 정규식을 작성하는데 걸리는 시간이 아까웠고, `루트`의 이야기를 듣고 스프링 자체적으로 제공하는 파싱 방법이 있을꺼 같아서 찾아보게 되었다.

---

## AntPathMatcher

```java
private Long getRoomId(final String url) {
    String actualUrl = "/topic/rooms/{roomId}/chat";
    AntPathMatcher pathMatcher = new AntPathMatcher();

    Map<String, String> variables = pathMatcher.extractUriTemplateVariables(actualUrl, url);
    return Long.valueOf(variables.get("roomId"));
}
```

- AntPathMatcher 를 이용하면 쉽게 파싱할 수 있었다.
- 추출하는 정규식 코드를 작성할 필요가 없다.
- 원하는 형태의 `actualUrl`을 만들어두고, 들어온 url을 맞춰서 넣어주면된다.
- map으로부터 key값을 통해 원하는 문자열을 얻을 수 있다.

### 주의할점
- 실제로 들어오는 `url`이 `actualUrl` 형태와 같게 해야하므로, 앞에서 전처리나 분기처리가 필요하다.
- 인자를 넣는 순서에 주의해야한다. `(actualUrl, url)` 순서이다. 반대로 넣지 않게 주의한다.
- `@PathVariable Long id` 과 다르게, 문자열이 숫자형태라고 해서 자동으로 숫자타입으로 캐스팅되지 않으므로 String으로 매핑 해야한다.

```
Map<String, String> variables // ok
Map<String, Long> variables // No
```

---

## 코드

```java

import org.springframework.util.AntPathMatcher;

import java.util.Map;

public class UrlPhaser {
    public static void main(String[] args) {
        String url = "/topic/rooms/1/chat";
        Long roomId = getRoomId(url);
        System.out.println(roomId);
    }

    private static Long getRoomId(final String url) {
        String actualUrl = "/topic/rooms/{roomId}/chat";
        AntPathMatcher pathMatcher = new AntPathMatcher();

        Map<String, String> variables = pathMatcher.extractUriTemplateVariables(actualUrl, url);
        return Long.valueOf(variables.get("roomId"));
    }
}
```

```java
// result
1
```

---

## Reference
- https://stackoverflow.com/questions/26347868/spring-utility-for-parsing-uris
- https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/util/AntPathMatcher.html
- https://github.com/spring-projects/spring-framework/blob/main/spring-web/src/test/java/org/springframework/web/util/pattern/PathPatternTests.java