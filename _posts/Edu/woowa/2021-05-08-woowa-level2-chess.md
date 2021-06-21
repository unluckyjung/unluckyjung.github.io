---
title: 우아한 테크코스 Level2 스프링 입문 미션 회고
date: 2021-05-08-21:00
categories:
- Edu

tags:
- 우아한 테크코스
- 교육
- Spring
- Java

photos: 
- /post_images/woowa-logo2.jpg

---

## 스프링 입문 - 체스 스프링 적용하기
> 스프링 프레임워크 입문을 위한 미션

- Level1 때는 한 미션이 끝난 기점으로, 해당 미션을 진행하면서 공부한 내용을 정리하지 않았다.
- Level2 때는 앞서 [3월 회고록에 적은것](https://unluckyjung.github.io/my/2021/04/10/Retrospective-of-March/#%ED%8E%98%EC%96%B4%EC%9D%98-%EC%9E%A5%EC%A0%90-%EB%B2%A4%EC%B9%98%EB%A7%88%ED%82%B9) 처럼 한 미션이 완전히 마무리된 시점에서, 배운내용을 기록하려고 한다.
- 매일 적는 TIL 내용 + 학습로그를 기반으로, 복습겸 블로그에 전체적인 학습내용을 되돌아보며 기록 해본다.

---

## Spring MVC 기초학습
- 브라운의 [테스트로 배우는 Spring 학습 테스트](https://github.com/next-step/spring-learning-test) 레포지토리를 이용한 학습
- [전체적인 내용 요약](https://gist.github.com/unluckyjung/bc28feb8c2edf28d3d70e104ccdd10cb)

---

## Controller vs RestContoller

```java
@RestController
@RequestMapping("/room")
public class SpringChessApiController {

    private SpringChessService springChessService;

    public SpringChessApiController(SpringChessService springChessService) {
        this.springChessService = springChessService;
    }

    @PostMapping
    //@ResponseBody
    private Map<String, String> createRoom(@RequestBody RoomDto roomDto) {
        Map<String, String> result = new HashMap<>();
        result.put("result", "success");
        result.put("roomId", springChessService.create(roomDto));
        return result;
    }
}
```

- `@RestContoller` 어노테이션을 걸었다면, 메서드들 머리위에 `ResponseBody`를 넣을필요가 없다.
- RestContoller는 Contoller + ResponseBody 형태이다.
- Contoller는 view에 직접 뿌릴때 쓰이고, (xxx.html), RestContoller는 Json형태로 객체 데이터를 반환하는데 쓰인다!

### 참고 링크
- https://mangkyu.tistory.com/49
- https://javarevisited.blogspot.com/2017/08/difference-between-restcontroller-and-controller-annotations-spring-mvc-rest.html#axzz6sCjOqKMI)


---

## Spring 스키마 자동생성


```java
spring.datasource.url=jdbc:mysql://localhost:3306/mydb?createDatabaseIfNotExist=true

spring.datasource.schema=classpath:schema.sql
spring.datasource.initialization-mode=always
```

- `createDatabaseIfNotExist=true` 를 해주면된다.
- `classpath:` 도 반드시 같이 적어주어야 한다.
- 주의할점은 이 과정에서 mydb라는 이름을 가진 스키마가 생성되고, use까지 하게 된다.
  - 즉, `schema.sql`는 CREATE가 아닌 ALTER를 해주어야 한다.


```sql
ALTER DATABASE chess DEFAULT CHARACTER SET = utf8;

CREATE TABLE IF NOT EXISTS users (
    name varchar(255) NOT NULL PRIMARY KEY,
    win int(11) NOT NULL default 0,
    lose int(11) NOT NULL default 0
);
```

### 참고링크
- https://stackoverflow.com/a/51179508



---

## contentType vs accept

- contentType : **내가 보내는** 타입은 이거다!!
- accept는 : 나는 요청보냈고!, **내가 원하는 타입**은 이거다!!
- 서버에 요청하는 body에는 항상 ContentType이 필요하다
- body에 무언가를 담아서 요청을 보낼때, ContentType이 무엇인지 명명해주면서 보내야한다.

```java
.contentType(MediaType.APPLICATION_JSON_VALUE)
.accept(MediaType.APPLICATION_JSON_VALUE)
```
---

## 동일성이 아닌 동등성을 비교하기

- 테스트하려는 객체에서 매번 get을 하지 않지 않고도 내부의 값들을 기반으로 테스트할 수 있다.

```java
@DisplayName("room에 존재하는 유저들의 전적을 요청하면, 각 userName별 전적을 반환한다.")
@Test
void usersInRoom() {
    UsersInRoomDto expectedUsersInRoomDto = new UsersInRoomDto(
            "fortune", "0", "0",
            "portune", "0", "0"
    );
    UsersInRoomDto usersInFirstRoomDto = springChessService.usersInRoom("1");
    assertThat(usersInFirstRoomDto).usingRecursiveComparison().isEqualTo(expectedUsersInRoomDto);
}
```

- 1번방으로부터 얻는 DTO가 예상될때 `expectedUsersInRoomDto.getName()` 와 같이 매번 꺼내와서 비교하지 않고
- `usingRecursiveComparison()` 를 통해서 한번애 동등성을 비교할 수 있다.
- 이 외에도 어느 한 필드만 제외하고 테스트하기, List를 비교한다면 담겨져 있는 순서와 관계없이 테스트하기 등등 여러 가지 기능이 있다.
  - [공식문서](https://assertj.github.io/doc/#assertj-core-recursive-comparison)를 참고하자.

### 참고 링크

- https://assertj.github.io/doc/#assertj-core-recursive-comparison
- https://www.podo-dev.com/blogs/232
- https://woowacourse.github.io/javable/post/2020-11-03-assertJ_methods/


---

## Spring Boot 테스트하기
- 통합 테스트, 유닛테스트, 기능 테스트의 구분

### 참고링크
- https://cheese10yun.github.io/spring-boot-test/
- https://cjwoov.tistory.com/9
- https://codeutopia.net/blog/2015/04/11/what-are-unit-testing-integration-testing-and-functional-testing/

---

## @AutoWired 어노테이션 생략하기
- 생성자가 하나밖에 없는경우 생략해줄 수 있다.
- `@TestConstructor(autowireMode = TestConstructor.AutowireMode.ALL)` 클래스단에 적어주면, @Autowired 어노테이션을 생략할 수 있다.


### 참고링크
- https://github.com/woowacourse/jwp-chess/pull/254#discussion_r615266892
- https://docs.spring.io/spring-framework/docs/5.3.5/javadoc-api/org/springframework/test/context/TestConstructor.html


---

## @componentscan과 Bean 주입
- Component 
  -  이거 Bean으로 쓸꺼야! 를 스프링에게 알려준다.
  - 즉 `@Service`, `@Contoller`, `@Repository` 전부 다 Component 어노테이션을 가지고 있다.

- ComponentScan    
  - `@SpringBootApp` 어노테이션이 가지고 있다.
  - 스캔을 쭉 하면서 어떤걸 Bean으로 넣어야할지를 알게된다
  - 스캔하는 범위는 자기 계층 + 하위 계층(패키지 구조상)을 스캔한다.

- @AutoWired 의존성 주입방법
  - 생성자 머리위에 어노테이션을 붙여주면 된다.
  - 필드에 선언해도 적용이 된다.
  - 생성자 + 필드 동시에 적용해도 동작이 된다.
  - 생성자의 매개변수로 필드객체를 주입받는 경우 **생략이 가능하다**

### 참고링크
- https://atoz-develop.tistory.com/entry/Spring-Component-%EC%95%A0%EB%85%B8%ED%85%8C%EC%9D%B4%EC%85%98-%EB%B0%8F-%ED%95%A8%EA%BB%98-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94-%EC%95%A0%EB%85%B8%ED%85%8C%EC%9D%B4%EC%85%98-%EC%A0%95%EB%A6%AC
- https://galid1.tistory.com/512

---

## @responsebody의 역할
- HTTP 요청으로온 Body를 자바 객체로 변환시켜서 받을 수 있다. **(수신)**
- HTTP 응답 데이터로 보낼 수 있다. **(송신)**


### @RequestBody 활용하여 메서드 파라미터로 활용하기

```java
@RestController
@RequestMapping("/method-argument")
public class MethodArgumentController {
    @PostMapping("/users/body")
    public ResponseEntity requestBody(@RequestBody User user) {
        User newUser = new User(1L, user.getName(), user.getEmail());
        return ResponseEntity.created(URI.create("/users/" + newUser.getId())).body(newUser);
    }
}
```

```java

```java
public ResponseEntity requestBody(@RequestBody User user)
```

```java
.body(user)
.when().post("/method-argument/users/body")
```

- 요청을 처리할 수 있게 된다.
- body안에 user가 담긴 정보를 받을 수 있게 된다.


### @ResponseBody 로 body에 데이터 담아 응답하기

```java
@Controller
@RequestMapping("/return-value")
public class ReturnValueController {

    @GetMapping("/message")
    @ResponseBody
    public String string() {
        return "message";
    }

    @GetMapping("/users")
    @ResponseBody
    public User responseBodyForUser() {
        return new User("name", "email");
    }
}
```

```java
@ResponseBody
```

```java
.when().get("/return-value/message")
.when().get("/return-value/users")
```

- body의 내용만 필요한경우, get에 대한 요청을 아주 쉽게 소화할 수 있게된다.
- 자바 빈 규약을 지켰다면, `객체 형식(User)`으로 보내주어도 데이터가 잘 담겨서 간다. DTO 처럼 쓰이는것이다.
  - 다르게 말하면 DTO로 또 옮겨담을 필요가 없다면, 도메인 객체를 재활용 할 수 있다.
  - 그래도 구분을 두기위해 같은 형태의 DTO를 또 만들어주어야 할까? 라는 의문이 있다면
  - [자바지기님의 글](https://www.slipp.net/questions/22#answer-121) 을 읽어보자

### 참고링크
- https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/ResponseBody.html
- https://devbox.tistory.com/entry/Spring-RequestBody-%EC%96%B4%EB%85%B8%ED%85%8C%EC%9D%B4%EC%85%98%EA%B3%BC-ReponseBody-%EC%96%B4%EB%85%B8%ED%85%8C%EC%9D%B4%EC%85%98%EC%9D%98-%EC%82%AC%EC%9A%A9
- https://www.slipp.net/questions/22#answer-121

---

## TRUNCATE
- 테이블의 데이터를 삭제할때 용이하다.
- 하지만, AutoIncrement로 늘어나는 값들은 초기화가 되지 않는다.


### 참고 링크
- https://goddaehee.tistory.com/55

---

## SELECT으로 특정값이 있는지 여부를 boolean으로 처리하기

### 1번방법
- 매칭되는것을 카운트 하고 분기처리

### 2번방법
- 예외를 잡아 처리

### 3번 방법
- 서브쿼리를 이용해 NOT EXISTS를 이용하기
- 이 방법을 채택했다.

```sql
public boolean isNotExist(String userName) {
    String query = "SELECT NOT EXISTS(SELECT * FROM users WHERE name = ?)";
    return jdbcTemplate.queryForObject(query, Boolean.class, userName);
}
```

### 참고링크
- [1,2 번 방법](https://stackoverflow.com/questions/18503607/best-practice-to-select-data-using-spring-jdbctemplate/18503994)

---

## @Component, @Configuration
- 내부를 들여다보면, @Configuration안에 @Component를 가지고 있다.
- 이 어노테이션을 통해 Spring에게 이놈도 Scan해야해! 를 알려주는것 같다 `2021-04-26`
  - 둘이 동작방식이 조금 다르다고 한다. [정리한 내용](https://unluckyjung.github.io/spring/2021/06/11/Spring-Configuration-VS-Component/)


---

## testCode 생성자는 테스트별로 매번 따로 돈다.
- 테스트가 돌떄마다 매번 주입을 새로 받는다.
- 즉 테스트 클래스내부에 생성자가 있고, 테스트가 8개 라면 생성자는 총 8번 호출된다.

---

## POJO vs Java Bean
- 모든 Bean은 POJO이나, 모든 POJO는 Java Bean이 아니다.
- Bean이 좀더 제한이 엄격한 특별한 POJO이다.

### 참고링크
- https://happyer16.tistory.com/entry/POJOplain-old-java-object%EB%9E%80
- https://www.geeksforgeeks.org/pojo-vs-java-beans/

---

## 생성자 주입 vs 필드 주입 vs setter 주입
- 생성자 주입은 **빌드할때 실패하기 때문에 순환참조가 발생하지 않는다!**
- 생성자 주입을 선호 하는 이유는, final 처리, null 처리 등등이 있지만, 순환 참조에 대한 대응이 쉽다.
- 세터 주입이나 필드 주입은 순환참조가 발생한 걸 빌드 단계에서 알지 못하지만, 생성자 주입은 미리 알고 수정할 수 있다.
- 이 순환참조를 해결하기 위한 방법으로는, 아예 설계를 다시하거나 lazy 등등 여러 방법이 있고, 권장되진 않지만 Setter 주입으로 변경해서 처리할 수 있다.

### 참고링크
- https://www.baeldung.com/circular-dependencies-in-spring
- https://yaboong.github.io/spring/2019/08/29/why-field-injection-is-bad/
- https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-setter-injection

---

## Component vs Bean
- Bean의 경우에는 외부라이브러리를 사용하는데 쓰이는 정도로 이전에 학습한적이 있다.
- 당시에는 관례정도로 생각하고 넘어갔는데, 이번에 BeanConfig을 따로 분리하면서 선언될 수 있는 타입이 나누어져 있는것을 학습했다.
- 즉 개발자가 직접 만든 Class는 @Bean 선언이 불가능하다

---

## @SQL
- 테스트를 실행하기전에 sql 스크립트를 실행하게 할 수 있다.
- 롤백 기능보다 아예 테이블 자체를 지웠다가 만드는 작업 + Insert를 수행하고 싶을떄 용이하게 사용할 수 있다.
- 여러 스크립트를 돌리는것도 가능하며, 클래스 머리위에 달아줄경우 모든 테스트마다, 
- 테스트 메소드위에 달아줄경우 해당 테스트를 수행하기전에 스크립트가 실행된다.

### 참고링크
- https://www.baeldung.com/spring-boot-data-sql-and-schema-sql
- https://javacan.tistory.com/entry/spring41-AtSql-annotation-intro

---

## jar build시 test코드 수행 생략
- 기본적으로 gradle build시 테스트도 다 진행하게 되는데, 아래의 구문으로 테스트를 생략할 수 있다.

```
gradle build -x test`
```

### 참고링크
- https://stackoverflow.com/questions/4597850/gradle-build-without-tests
- https://devday.tistory.com/entry/Gradle-%ED%85%8C%EC%8A%A4%ED%8A%B8-Test-%EC%8A%A4%ED%82%B5%ED%95%98%EA%B8%B0-Skip

---

## Bean을 Config 파일로 만들어 따로 관리하기
- [Bean을 Config 파일에서 정의해 관리하기](https://unluckyjung.github.io/spring/2021/05/30/Spring-BeanConfig/)

---

## @Bean vs @Component
- [Bean vs Component](https://unluckyjung.github.io/spring/2021/06/06/Spring-Bean-VS-Component/)

---