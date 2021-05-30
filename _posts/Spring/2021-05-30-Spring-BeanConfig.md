---
title: Bean을 Config 파일에서 정의해 관리하기
date: 2021-05-30-21:00
categories:
- Spring

tags:
- Spring
- Bean

---

## Bean을 Config 파일에서 정의하기
> 빈 관리가 편해진다.

---

## Goal
- 외부 라이브러리인 Gson을 `@Bean` 어노테이션을 이용해 관리한다.
- **이때 Config 파일을 만들어 빈 정의를 한곳에서 관리한다.**

---

## 기존의 코드
> `Repository.class` 에서 @Bean을 이용해 Gson을 빈으로 등록

```java
@Repository
public class PlayLogRepository {
    private static final Gson GSON = new Gson();
    ...
}
``` 

- `PlayLogRepository` 에서 Gson 라이브러리를 사용하기 위해, Gson을 선언하고 사용하고 있었다.  
  - 재사용이 가능한 자원이므로, static 하게 관리했었다.



```java
@Repository
@Configuration
public class PlayLogRepository {

    private final JdbcTemplate jdbcTemplate;

    public PlayLogRepository(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    @Bean
    public Gson gson(){
        return new Gson();
    }
    ...
}
```

- 그러다가 `Bean`으로 등록하면 아예 스프링에서 관리할 수 있다는 것을 깨닫고 위와같이 코드를 변경했었다.

![image](https://user-images.githubusercontent.com/43930419/115986528-8f006f00-a5eb-11eb-93ed-4ef993a6f133.png)

- 그러나 위의 코드에 대해서 다음와 같은 리뷰를 받았다.  

```
@Bean을 이용해 빈을 만들면 다른 컴포넌트 들에서도 쓸 수 있기 때문에 별도로 Config 파일을 만들어주고 있어요.
config 패키지 하위에 BeanConfig 또는 JsonConfig 를 만들고 거기서 Gson을 빈으로 정의해주세요 :)
```

- 리뷰만 봤을때 `@Bean` 어노테이션을 쓰지 말라는것인가? 하는 고민을 엄청나게 했었다.
- 자세히 읽어보니 다른곳에서 Bean을 정의하고 주입받으란 코멘트였다.



## 수정한 코드

```java
//BeanConfig

package chess.config;

import com.google.gson.Gson;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class BeanConfig {

    @Bean
    // 메소드 명은 반환타입만 올바르다면, 아무렇게나 해주어도 괜찮다.
    // public Gson yoonSung() 역시 괜찮다. 
    public Gson gson() {
        return new Gson();
    }
}
```

- 그리고 이걸 기존 Repository에서 어떻게 사용하지? 라는 고민을 했다.
- 방법은 간단했다.

```java
@Repository
public class PlayLogRepository {

    private final JdbcTemplate jdbcTemplate;
    private final Gson gson;

    // 처음에는 gson을 넘겨 받는것이 아니라 BeanConfig을 넘겨받아야 하나? 고민했다.
    public PlayLogRepository(JdbcTemplate jdbcTemplate, Gson gson) {
        this.jdbcTemplate = jdbcTemplate;
        this.gson = gson;
    }
}
```

- 이렇게 주입받을 수 있게된다..!!

---

## Config 파일로 관리하는 이유
- 굳이 왜 Config 파일을 따로 만들어서 Bean을 정의할까?
- 어차피 `@ComponentScan` 이 알아서 빈을 잘 찾아서 넣어줄텐데?
  - Gson을 빈으로 만들어 주었다는것을, 한 파일에서 보고 관리할 수 있기 때문이다.

- 당장은 `PlayLogRepository` 한 곳에서만 Gson을 사용하고 있지만, 나중에 다른곳에서도 Gson이 필요하다면?
  - Bean이 `PlayLogRepository`에서 정의 되어있는것을 모르고 불필요한 재정의가 이루어질 수 도 있다.
  - 불필요한 코드가 늘어나는것이다.

---

## Reference
- https://galid1.tistory.com/494
- https://cho1-w0n-san9.tistory.com/23
