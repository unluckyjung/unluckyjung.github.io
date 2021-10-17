---
title: Component vs Configuration
date: 2021-06-11-16:00
categories:
- Spring

tags:
- Spring
- Bean
- Annotation

---

## @Component와 @Configuration 의 차이를 알아봅니다.
> Bean을 반환할때 기능상 비슷하게 동작하나, 다르게 동작합니다.

---

## Goal
- `@Component` 와 `@Configuration` 의 차이를 알아본다.
- 어떤 어노테이션을 쓰는것이 좋을지 고민해본다.


---

## @Configuration 들여다보기

![image](https://user-images.githubusercontent.com/43930419/121636215-880dbd00-cac2-11eb-90fd-94e5e4a83fb9.png)

- `@Configuration`을 보면 `@Component` 를 들고 있는형태이다.
- 실제로도 `@Component`와 `Configuration` 를 서로 바꾸어주어도 **기능상으로는** 아주 비슷하게 동작한다.  
    - 둘다 `@Bean` 어노테이션이 붙은 메소드들을 찾아 스프링 컨테이너에 등록시키는 역할을 수행한다.

```java
@Configuration
public class WebConfigUseConfiguration {

    @Bean
    public BeanA beanA(){
        return new BeanA();
    }
}
```

```java
@Component
public class WebConfigUseComponent {

    @Bean
    public BeanA beanA(){
        return new BeanA();
    }
}
```

- `WebConfigUseConfiguration.class`, `WebConfigUseComponent.class` 둘다 **BeanA** 를 스프링 컨테이너에 등록시키는건 같다. 

---

## 스프링 컨테이너에 등록된 빈을 반환하느냐, 안하느냐
> 같은 Config 클래스내에서, 내부적으로 Bean을 등록하고, 사용하는경우

- 둘은 Bean을 반환할때 동작방식이 다르다.

```java
@Configuration
public class WebConfigUseConfiguration {

    @Bean
    public BeanA beanA(){
        return new BeanA();
    }

    @Bean
    public BeanB beanB(){
        // 위의 beanA() 메소드를 호출 중
        return new BeanB(beanA());
    }
}
```

- `beanB()` 의 경우, `beanA()` 메소드를 호출하려고한다.
- `@Configuration` 의 경우 `beanA()` 를 호출하면 스프링 컨테이너에 등록된 **BeanA**를 가져온다. 
- 하지만 `@Component`의 경우, 스프링 컨테이너에 등록된 **BeanA**를 가져오지 않고, `beanA()`를 호출해, **새로운 BeanA** 인스턴스를 가져오게된다.
  - **새로 생성된 빈이 반환되는 것이다.**
  - 싱글턴이 보장되지 않는다.


```java
@Component
public class WebConfigUseComponent {
    @AutoWired
    private BeanA beanA;

    @Bean
    public BeanA beanA(){
        return new BeanA();
    }

    @Bean
    public BeanB beanB(){
        return new BeanB(beanA());
    }
}
```

- 만약 `@Component`를 사용하는 경우에도 `@Configuration` 처럼 작동하게 하고 싶다면 위와 같이 수정해주면 된다.

---

## 가능하면 @Configuration을 사용하자
> @Configuration classes currently always get enhanced via CGLIB. 

- `@Configuration` 을 사용시 **CGLIB** 처리를 통해 성능 향상을 기대할 수 있다고 했다.
  - [스프링 개발자 오피셜](https://github.com/spring-projects/spring-framework/issues/17430#issuecomment-453423770)
  - 내부에서 호출된 `@Bean` 이 붙은 메소드를 호출하는 경우, 이미 등록된 Bean이 있으면 컨테이너에서 꺼낸다.
  - 없는 경우는 등록하고 반환한다! 

---

### 참고
- `@Bean`메소드를 들고 있는 클래스단에 `@Configuration`을 달아주지 않을 경우, 디폴트로 클래스단에 `@Component`가 붙어 돌아가게된다.

---

## Reference

- https://docs.spring.io/spring-framework/docs/4.0.4.RELEASE/javadoc-api/org/springframework/context/annotation/Configuration.html
- https://stackoverflow.com/a/48175776
- http://dimafeng.com/2015/08/29/spring-configuration_vs_component/
- https://m.blog.naver.com/sthwin/222131873998