---
title: WebClient vs RestTemplate
date: 2022-03-19-17:00
categories:
- Spring

tags:
- Spring
- WebFlux
- RestTemplate
- WebClient


---

## WebClient와 RestTemplate의 차이점을 알아봅니다.
> 기본적으로 블로킹 기반 vs 논블로킹 기반이냐의 차이가 있습니다

## Goal
- 스프링에서 API를 호출할때 사용하는 HTTP 클라이언트인 **WebClient**와 **RestTemplate**의 차이점을 알아봅니다.
- 둘중 어떤것을 사용하면 좋을지에 대해서 고민해봅니다.

---

## Synchronous and blocking(RestTemplate) vs All Support (WebClient)

### RestTemplate
> **Spring 3.0**, Blocking/Sync

- **RestTemplate**의 경우, 요청마다 스레드를 만드는 Java Servlet API를 사용합니다.
- 즉, 웹클라이언트가 응답을 받을때까지 Thread를 **Blocking** 하게 됩니다. 이로인해서 **동기/블로킹** 환경이 됩니다.
- 이로 인해서 요청이 굉장히 많은경우, **스레드풀이 부족**해지는 현상이 일어날 수 있고, 스레드가 과도하게 많이 생겨 잦은 **Context Switch**로 인해 **성능 저하**가 발생할 가능성이 있습니다.
- 비동기적인 프로그래밍이 필요한경우에는 **AsyncRestTemplate**를 사용할수 있습니다.

### WebClient
> **Spring5**, 기본적으로 **NonBlocking/Async** 이지만, Blocking, Sync 역시 같이 지원합니다. **이벤트 기반** 입니다.

- **WebClient**의 경우 기본적으로 `Spring Reactive FrameWork`에서 제공하는 **NonBlocking** 방법을 사용합니다. 
- `Spring Reactive FrameWork`의 경우 **이벤트 기반**으로 작동하기 때문에, 동기/블로킹 환경보다 **더 적은 자원**을 소요하는 구조를 가지게 됩니다. 이벤트가 발생시 **"task"** 를 만들게 되고 대기열에 넣어둔뒤, **"task" 를 처리할 수 있을때서야 작업을 처리**하기 때문입니다.
- **WebClient**에 대한 좀더 구체적인 내용은 사용법과 같이 따로 분리해 정리할 예정입니다. 

---

## More
> RestTemplate는 곧 사라질 예정입니다.  

> NOTE: As of 5.0 this class is in maintenance mode, with only minor requests for changes and bugs to be accepted going forward. Please, consider using the org.springframework.web.reactive.client.WebClient which has a more modern API and supports sync, async, and streaming scenarios.

- RestTempalate는 [docs](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html)에 따르면, 곧 사라질 예정입니다.
- 현재 유지보수만 진행중이고, 스프링팀에서도 **WebClient** 사용을 권장하고 있습니다.

---

## Conclusion
- `RestTemplate`는 Java Servlet API를 사용하므로 **블로킹/동기** 로 작동하게 된다.
- `WebClient`는 기본적으로 **논블로킹/비동기** 로 작동하나, **블로킹이나 동기** 역시 지원한다.
- `RestTemplate` 곧 **deprecated** 될 예정이기 때문에, 여유가 있다면 기존의 코드들은 `WebClient`로 마이그레이션하거나, 새로운 프로젝트들은 `WebClient`로 개발하는것이 좋아보인다.

---


## Reference
- [https://www.baeldung.com/spring-webclient-resttemplate](https://www.baeldung.com/spring-webclient-resttemplate)
- [https://stackoverflow.com/questions/47974757/webclient-vs-resttemplate](https://stackoverflow.com/questions/47974757/webclient-vs-resttemplate)
- [RestTemplate docs](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html)
- [WebClinent docs](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/reactive/function/client/WebClient.html)
- https://ddoriya.tistory.com/entry/RestTemplate-VS-WebClient