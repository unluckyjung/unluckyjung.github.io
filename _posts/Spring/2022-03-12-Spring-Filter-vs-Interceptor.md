---
title: Filter vs Interceptor
date: 2022-03-12-17:00
categories:
- Spring

tags:
- Spring
- Filter
- Interceptor
- ArgumentResolver

---

## Filter와 Interceptor 차이를 알아봅니다.
> 요청, 응답을 처리하는 시점이 다릅니다.


---

## Goal
- **Filter**와 **Interceptor**의 차이를 알아봅니다.
- Spring **ArgumentResolver** 와의 차이도 같이 알아봅니다.

---

## Servlet Filter vs Interceptor 
> 공통적인 작업을 처리해줍니다.

- 하지만 처리되는 시점과 관리되는 매체가 다릅니다.

### Filter
> **웹 컨테이너에** 의해서 관리됩니다. (스프링과 무관한 전역적인 처리작업을 수행합니다.)

- 디스페쳐 서블릿에 요청이 **전달되기 전** 처리합니다.
- 보안 관련 작업을 처리합니다. 모든 요청에 대한 로깅을 처리합니다. (다른 프레임워크로 변경해도, 보안관련 작업은 유지되어야 하기 때문입니다.)
- 모든 HTTP 요청에 대한 로깅이 필요한경우에 쓰입니다.

![image](https://user-images.githubusercontent.com/43930419/141442023-b1bb9f22-e120-4669-8184-2e1b9b95eb70.png)

  
### Interceptor
> 스프링이 제공하며 **스프링 컨테이너**에 의해서 관리 됩니다.

- 디스페쳐 서블릿이 컨틀롤러를 호출하기 **전,후** 에 대한 요청, 응답을 처리합니다.
- **인증/인가** 와 같은 작업에 대해서 처리합니다.
- 처리해야할 API 호출에 대한 로깅을 처리합니다.


![image](https://user-images.githubusercontent.com/43930419/141441978-54b81159-2a0e-48d3-b724-f740070ef682.png)

---

## Spring Argument Resolver
>  여러 핸들러에 **공통적으로 필요한 파라미터**(값 또는 객체)를 생성 해주는 모듈을 말합니다.


- 어노테이션 기반 컨트롤러를 처리하는 **RequestMappingHandlerAdaptor**는 **ArgumentResolver**를 호출합니다.
- **ArgumentResolver**는 **Controller(핸들러)** 가 필요로 하는 다양한 파라미터의 값(객체)를 생성해줍니다.
- 파라미터의 값이 모두 준비되면(파라미터 생성 완료 후) Controller 를 호출하면서 값을 넘겨주게 됩니다.

### 구현방법
- `HandlerMethodArgumentResolver`를 상속한 클래스를 만듭니다.

```java
// HandlerMethodArgumentResolver.class

boolean supportsParameter(MethodParameter parameter);

@Nullable
Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer, NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception;
```

- `supportsParameter` 메소드에 원하는 타입의 인자가 있는지를 확인해 `true/false` 를 반환합니다.
- `resolveArgument` 단계에서 반환할 객체를 명시합니다.
- 개발할때 많이 사용하는 `@RequestBody` 같은 경우도 ArgumentResolver를 통해 메시지 컨버터를 사용해 필요한 객체를 생성합니다. (이부분은 다른글로 분리해 작성할 예정)

### 장점
- 핸들러 **파라미터에 대한 공통작업**을 처리할때 편리합니다.

---

## Conclusion

![image](https://user-images.githubusercontent.com/43930419/158014048-400ac1fd-359e-409d-b5d2-aee0637b2b64.png)


- Filter, Interceptor, ArgumentResolver는 공통적인 작업을 처리하는데 사용됩니다.
- **Filter**는 DispatcherServlet 기준 **앞단**에서, **Interceptor**는 **뒷단**에서 작동합니다.

---

## Reference
- https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/HandlerInterceptor.html
- https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/method/support/HandlerMethodArgumentResolver.html
- https://supawer0728.github.io/2018/04/04/spring-filter-interceptor
- https://mangkyu.tistory.com/173
- https://tecoble.techcourse.co.kr/post/2021-05-24-spring-interceptor/