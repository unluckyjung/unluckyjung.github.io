---
title: Bean vs Component
date: 2021-06-06-23:00
categories:
- Spring

tags:
- Spring
- Bean
- Annotation

---

## @Bean과 @Component의 차이를 알아봅니다.
> 외부 라이브러리를 빈으로 등록하느냐, 직접 만든 클래스를 빈으로 등록하느냐

---

## Goal
- Bean과 Component의 차이를 알아봅니다.

---

## Component
- 클래스단 위에서 선언한다.
- 개발자가 직접 작성한 Class 파일을 Bean 으로 등록할때 쓰인다.

## Bean
- 메소드 위에서 선언한다.
- 외부 라이브러리들을 Bean 으로 등록할때 쓰인다.

---

## 달수 있는 위치의 차이

![image](https://user-images.githubusercontent.com/43930419/120911573-6a57e680-c6c3-11eb-87ee-075677e62575.png)

![image](https://user-images.githubusercontent.com/43930419/120911566-5c09ca80-c6c3-11eb-9b7b-57d769cf03f8.png)


```java
// @Component
@Target(ElementType.TYPE)


// @Bean
@Target({ElementType.METHOD, ElementType.ANNOTATION_TYPE})
```

- 각자 선언될수 있는 타입이 정해져있다.
- `@Bean` 은 Method에서만, `@Component`는 Class 위에서만 선언이 가능하다.


```java
@Component
public class BeanConfig {

    @Bean
    public Gson gson() {
        return new Gson();
    }
}
```

- 이전에 작성한 코드를 다시보자.
- 외부라이브러리인 **GSON**을 빈으로 등록하기 위해 리턴하는 gson() 메소드위에 `@Bean` 어노테이션을 달아주었다.
  - 왜 `@Component`가 아닌 `@Bean`을 달아줄까?   

```java
@Component
public class Gson {
    ...Gson에 대해서 다시 구현할래..?
}
```

- `@Component` 는 내가 만든 클래스를 Bean 으로 등록한다. 
- 그런데 `Gson`을 Component로 관리한다는 소리는 내가 `Gson` 라이브러리를 쓰는것이 아니라, 직접 구현한다는 소리와 마찬가지이다.

---

## TIP
- `@Component`를 `@Configuration`로 바꿔주어도 똑같이 동작하는것처럼 보인다.
  - `@Configuration` 가 `@Component`를 들고 있는 구조이다.
  - ~~하지만 동작 방식이 조금 다르다. 이부분에 대해서는 추가적으로 포스팅을 한뒤 링크를 걸 예정~~
  - [스프링 빈 싱글턴 보장 유무가 다르다.](https://unluckyjung.github.io/spring/2021/06/11/Spring-Configuration-VS-Component/)


---

## Reference

- https://stackoverflow.com/questions/10604298/spring-component-versus-bean
- https://docs.spring.io/spring-framework/docs/4.0.4.RELEASE/javadoc-api/org/springframework/context/annotation/Configuration.html
- https://goodgid.github.io/Spring-Component-vs-Bean/
- https://jojoldu.tistory.com/27