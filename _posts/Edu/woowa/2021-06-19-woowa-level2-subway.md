---
title: 우아한 테크코스 Level2 지하철 3종세트 미션 회고
date: 2021-06-19-15:00
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

# 지하철 노선도 미션
> 경로조회, 관리, 협업 까지  


level2의 미션의 도메인은 지하철 노선이 절반이상을 차지했고, 배운내용들은 요악하자면 아래와 같다.  

- 기본적으로 주어지는 뼈대코드를 바탕으로 살을 덧붙이는 경험을 해보았다.
- atdd를 경험해보았다.
- 스프링부트에 대해 학습했다.
- **한정되고 유한한 시간내에, 결과물을 산출하기 위해서 새로운것을 빠르게 학습하는 방법을 나 혼자 스스로 찾아갔다.**

볼드 표시한 부분이 프로그래밍 기술적인 부분이 아니지만, **Level2** 를 경험하면서 제일 중요하게 배워간것이라고 생각한다.  
이부분에 대한 회고는 다른 글로 분리해서 작성할 예정이다.  

---

# Atdd, Test Code

## RestAssured를 이용해 body 데이터를 테스트하는 방법

### given()함과 동시에 테스트 하는방법

```java
RestAssured.given()
.when().get("testUrl")
.then().body("key", is("value"));
```

- `hamcrest` Is를 이용해 테스트하는 간략한 예시이다.
- 바로 `then().body()` 를 이용해 테스트할 수 있다.

### given when then 단계를 구분하기 위해 AssertThat을 이용하는 방법

```java
private ExtractableResponse<Response> response = RestAssured.given().log().all().body(params)

// jsonPath
assertThat(response.response().jsonPath().getLong("id")).isEqualTo(1L);

// as
assertThat(response.as(LineResponse.class).getName()).isEqualTo("신분당선");
```

- 분리하는경우, 바로 body()를 통해서 테스트 하는것이 불가능하다.

#### jsonPath() 를 이용해 테스트하는 방법
- `response().jsonPath().getType("key")` 
- response의 body에 담겨있는 value값을 얻어와 테스트할 수 있다.

#### as를 이용해 테스트하는 방법
- `response.as(DTO.class).getName();` 
- 이 방법은 리플랙션을 이용해서 테스트하게 된다.
  - 사용하는 DTO에 디폴트 생성자를 직접 작성해주어야 한다.


- https://github.com/rest-assured/rest-assured/wiki/Usage
- https://www.baeldung.com/rest-assured-tutorial

---

## @NullAndEmptySource

어노테이션을 이용해 인자에 null이랑 empty를 넣어보며 쉽게 테스트할 수 있다.

```java
@ParameterizedTest
@NullAndEmptySource
@ValueSource(strings = {"에", "예예", "우아한테크코스검프에어바다포츈우기화이팅짱"})
void createStationFail(String name) {
    //when
    ExtractableResponse<Response> response = 지하철역_생성_요청(name, tokenResponse);

    // then
    지하철역_생성_실패됨(response);
}
```

---

## @MethodSource

`@Csvsource` 로 하기에 조금 복잡하고, 테스트별로 중복되는 값을 넣어보는걸 희망하는 경우에는  
`@MethodSource` 를 통해서 인풋, 아웃풋이 여러개인 테스트를 손쉽게 할 수 있다.  

```java
public class AssertJTest1 {
    private static int getPow(int number) {
        return number * number;
    }

    private static Stream<Arguments> powByNumber() {
        return Stream.of(
                Arguments.of(3, 9),
                Arguments.of(4, 16)
                Arguments.of(5, 25)
        );
    }

    @ParameterizedTest
    @MethodSource("powByNumber")
    void test(int number, int pow) {
        assertThat(getPow(number)).isEqualTo(pow);
    }

    @ParameterizedTest
    @MethodSource
    void powByNumber(int number, int pow) {
        assertThat(getPow(number)).isEqualTo(pow);
    }
}
```

- 주의할점은 반드시 `@MethodSource`에 사용될 method는 static 한 메소드여야하며 stream.of 형태로 반환해주어야한다.
- 테소드 메소드 이름을 동일하게 사용하면, 메소드 이름을 명시적으로 적어주지 않아도 자동으로 값이 들어간다.
  - `powByNumbr(int number, int pow)` 를 보면, `@MethodSource`에 명시해주지 않은것을 볼 수 있다.

**언제 사용하는게 좋을까?**  
> 리뷰어 : 객체를 만들어서 파라메터로 넘겨야 할때 사용한다. 그외의 상황에 대해선, 개인적인 생각으로는 `csvSource` 같은걸 사용하는게 더 좋아보인다!

---

# Srping Boot

## JDBC Template

### RowMapper

```java
private final RowMapper<Line> lineRowMapper = (resultSet, rowNum) -> new Line(
        resultSet.getLong("id"),
        resultSet.getString("color"),
        resultSet.getString("name")
);

public List<Line> getLines() {
    String query = "SELECT id, color, name FROM line ORDER BY id";
    return jdbcTemplate.query(query, lineRowMapper);
}
```

JDBCTemplate에서 제공하는 RowMapper를 이용하면, 쿼리에 걸리는 경우를 가지고 객체를 형태로 만든뒤 List에 담을 수 있었다.  
그런데 Mapping되는 형태가 굉장히 메소드처럼 생겨있어서 무의식적으로 메소드 위치에 내려놨었는데 주의하자.  
행 단위로 매핑하는데 사용되는 ResultSet 인터페이스이다.  
예외 처리역시 고민하지말자. JdbcTemplate에 의해 포착되고 SQLExceptions으로 처리가 알아서 잘된다.

- https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jdbc/core/RowMapper.html


---

## CommandLineRunner, ApplicationRunner, @Order
스프링부트 구동시에 특정 코드들을 동작시키게 할 수 있다.    
해당 인터페이스를 구현해둔 클래스위에 `@Component`를 붙여주면된다.  

둘의 차이는  
- 입력인자가 존재하는경우 타입을 String으로 받느냐, ApplicationArguments 차이일 뿐이다.
- 여러 동작이 일어나는것이 필요한 경우, 여러개를 등록할 수도 있다.
- 대신 순서를 `@Order(count)` 어노테이션을 통해 정해주어야 한다.

```java
@Order(1)   // 첫번째 순서
@Component
public class DataLoader1 implements CommandLineRunner {
    private final FooDao fooDao;

    public DataLoader(final FooDao fooDao){
        this.fooDao = fooDao;
    }

    @Override
    public void run(String... args) throws Exception {
        // 초기 데이터 세팅 과정
        fooDao.insert("초기 데이터 1");
    }
}


@Order(2)   // 두번째 순서
@Component
public class DataLoader2 implements ApplicationRunner {

    @Override
    public void run(ApplicationArguments args) throws Exception {
        // CommandLineRunner 은 String으로 입력받았으나, 
        // 이친구는 String을 한단계 더 추상화한 ApplicationArguments 타입으로 받는다.
    }
}
```


---

## @Profile, @ActiveProfile
프로필을 설정하여 Test를 용이하게 할 수 있다.  
스프링 3.1 이상부터 사용할 수 있는 기능이다.   

`@ActiveProfile` 을 이용해 테스트 클래스의 프로필을 설정해주고,  
`@Profile`을 이용해 테스트때 빈으로 등록하는것을 원치 않는 경우를 걸러낼 수 있게된다. 


```java
@ActiveProfiles("test")
public class AcceptanceTest {
    @LocalServerPort
    int port;

    @BeforeEach
    public void setUp() {
        RestAssured.port = port;
    }
}
```

```java
@Bean
@Profile("!test")
HelloService fooService() {
  return new fooService();
}
```

AcceptanceTest를 수행하면 `fooService` 는 빈으로 등록 되지 않는다.
test 프로필이 아닌경우에만 `fooService` 를 빈으로 등록할 수 있게된다.  

여러 프로필을 등록할 수 도 있다.  

```java
@Bean
@Profile({"!line", "!holiday"})
HelloService fooService() {
  return new fooService();
}
```


- http://wonwoo.ml/index.php/post/1933
- https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/env/Profiles.html#of-java.lang.String...-


---

## @Size vs @Length
둘의 기능은 동일하게 작동한다.  
`@Length`의 경우는 Hibernate 특정 버전이고, `@Size`의 경우 Bean을 JPA, Hibernate같은 벤더로부터 독립적으로 구성해준다.  
가능하면 `@Size` 를 사용하도록 하자.


---

## @ExtendWith(MockitoExtension.class)

```java
//@ExtendWith(MockitoExtension.class)
public class PathCalculatorTest {
    @Mock
    private SubwayPath subwayPath;

    @Test
    @DisplayName("나이가 어린이(6살~12살)이고, 노선 추가요금이 500원이며, 이동거리가 10km 이하이면, (기본요금 + 500원 - 청소년할인(350원) )에서 50퍼 할인 금액이 나온다. ")
    void ageTest() {
        given(subwayPath.getDistance()).willReturn(10);
        given(subwayPath.getMaximumExtraFareByLine()).willReturn(500);

        assertThat(FareCalculator.calculateFare(subwayPath, 6))
                .isEqualTo((int) ((FarePolicy.DEFAULT_FARE.getFare() - FarePolicy.MINOR_DISCOUNT_FARE.getFare() + 500) * 0.5));
    }
}
```

```
java.lang.NullPointerException
	at wooteco.subway.path.domain.fare.PathCalculatorTest.ageTest(PathCalculatorTest.java:21)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
```

- `@ExtendWith(MockitoExtension.class)`가 없으면 mocking 과정에서 예외가 발생한다.

---

## Mockito 에서 when과 given의 차이

```java
@ExtendWith(MockitoExtension.class)
public class PathCalculatorTest {
    @Mock
    private SubwayPath subwayPath;

    @Test
    @DisplayName("테스트")
    void ageTest() {
//        when(subwayPath.getDistance()).thenReturn(6);
        given(subwayPath.getDistance()).willReturn(6);

        assertThat(subwayPath.getDistance()).isEqualTo(6);
    }
}
```
- when도 가능 given도 가능
- given이 좀 더 의미상 명확하다는 이야기가 있다.


---


# WEB, 설계

## DTO를 들고 가는 레이어는 어디까지에 대한 고민
**DTO**를 Domain으로 변환하는 작업은 **서비스**에서 하는것으로 일단은 결론 지었다.

컨트롤러에서 변환하지 말아야 할 이유는 크게 2가지로 결론 지었다.  

1. DTO를 컨트롤러에서 도메인 객체로 변환한다면 클라이언트에게 필요하지 않은 데이터가 컨트롤러까지 올라오게 되고, 이것은 잘못된 설계라고 생각한다.  
2. DTO가 한개의 도메인 객체가 아닌 다수의 도메인객체를 이용해서 만들어진다면, 컨트롤러에서 도메인을 조합하는 역할을 수행하게 되므로 컨트롤러가 로직을 들고 있게 된다.  

- https://www.petrikainulainen.net/software-development/design/understanding-spring-web-application-architecture-the-classic-way/
- https://github.com/HomoEfficio/dev-tips/blob/master/DTO-DomainObject-Converter.md


---

## 중복된 Name, 없는 ID에 대한 적절한 응답코드

### 중복 Name에 대한 예외
`403 Forbidden`는 **인가**에 실패한경우에 쓰인다.  
`409 Conplict`는 대상 리소스가 현재 서버에 존재하는 리소스와 **충돌** 이 발생했음을 알려준다.  
즉, 중복된 Name에 대한 예외는 `409 Conplict`가 어울린다는 결론을 내렸다.  

- https://deveric.tistory.com/62

### 없는 ID에 대한 예외
`204 NoConent`는 **해당 정보가 없어!** 라는것을 의미하기 보다는, **너 굳이 다른 페이지로 넘어갈 필요가 없어** 를 이야기 해준다.  
그리고 성공 코드를 보내는거 자체가 문제가 있다고 생각한다.  
`404 notFound` 대표적인 예시로 존[재하지 않는 id](https://github.com/woowacoursejjang)로 깃헙에 접근하면 404에러가 난다.  
없는 데이터에 접근하므로 `NOT FOUND`로 결정했다.  

- https://medium.com/@laeshiny/get-%EC%9A%94%EC%B2%AD-%EC%8B%9C-%EB%8D%B0%EC%9D%B4%ED%84%B0%EA%B0%80-%EC%97%86%EC%9C%BC%EB%A9%B4-200-or-404-4ab7430084af

---

## 서비스가 다른 서비스나 여러 레포지토리를 접근해도 되는가?
못할건 없다. 중복되는 코드를 줄일 수 있다는 명확한 장점이 존재한다.  
하지만 참조는 한쪽에서만 해야한다. 항상 순환참조를 주의하자.  

- https://www.inflearn.com/questions/21703
- https://stackoverflow.com/questions/51988182/spring-boot-service-class-calling-another-service-class 

---

## 패키지 구조를 어떻게 가져갈까?

```
1안 - layer 우선

domain.modulename
service.modulename
repository.modulename

2안 - 모듈 우선
modulename.domain
modulename.service
modulename.dao
modulename.web
```

1안으로 하다가 2안으로 바꿨는데 굉장히 개발 피로도가 높아진다.  
그냥 레이어별로 구분하는게 더 좋을것 같다.  
나중에 입사한뒤 팀 컨벤션에 따르겠지만.. 두개다 경험을 해본결과 1안이 좀더 찾아가기도 편하고 좋았다.

---

## localStorage object save
로컬 스토로지에 object타입으로 저장이 불가능하다!  
즉 반드시 String 형태로 저장해야한다.  
만약 String 형태로 불러오고, Object 형태로 다시 바꾸고 싶다면

```js
//save
JSON.stringify(key, value);

// localStorage.setItem(TODOS_LS, toDos)
localStorage.setItem(key, JSON.stringify(object))


// load
JSON.parse(localstorage key);

const object = localStorage.getItem(key)
if (object !== null) {
  const realObject = JSON.parse(object);
}
```


- https://studyingych.tistory.com/28

---

## 외부 라이브러리의 기능도 DI 시키기
- [외부 라이브러리를 열어보고 추상화한뒤 dependency injection 시키기](https://unluckyjung.github.io/dev/2021/06/05/Library-DI/)

---


# 인증과 인가

## payload에 어떤 정보를 담을지에 대한 고민
![image](https://user-images.githubusercontent.com/43930419/118394346-19823e80-b67f-11eb-955b-65fd9f08f21c.png)

기존에 구현한 코드는 로그인할때 쓰이는 id인 email을 payload에 담아서 토큰을 생성했다.  
하지만 이방법은 email은 변경이 가능한 요소이기 때문에, 완전히 고유값을 가진 id를 사용하는것을 추천받았다.  

```java
public TokenResponse createToken(final TokenRequest tokenRequest) {
    if (checkInvalidLogin(tokenRequest.getEmail(), tokenRequest.getPassword())) {
        throw new AuthorizationException("등록되지 않은 사용자입니다.");
    }
    Long id = memberDao.findIdByEmail(tokenRequest.getEmail());
    return new TokenResponse(jwtTokenProvider.createToken(String.valueOf(id)));
}
```

- 토큰을 생성할때 Payload에 db안에 존재하던 id를 담도록 했다.
- 그리고, 유저의 정보가 변경될떄는 토큰을 재발급하는게 옳다고 생각했고, 맞다고 답변받았다.

---

## Interceptor을 이용해 특정 url을 필터링하기

```java
@Configuration
public class AuthenticationPrincipalConfig implements WebMvcConfigurer {

    ...

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginInterceptor(authService))
        .addPathPatterns("/**")
        .excludePathPatterns("/members", "/login/token");
    }
}

```

인터셉터에서 전체 url에 대해서 토큰 검사를 진행한다.  `addPathPatterns`
이때 처음 회원가입 관련된 부분에서는 token검사가 필요하지 않다면, `excludePathPatterns`를 이용해 특정 url에 대해서는 인터셉터를 건너뛰게 할 수 있다.

---

## javaB 는 java11 이상부터는 디폴트로 포함되어있지 않다.

올바른 요청이 왔음에도 불구하고, access token을 제대로 반환하지 못하는 상황이 발생했다.
실패 에러 메시지는 아래와 같았다.  


`"Handler dispatch failed; nested exception is java.lang.NoClassDefFoundError: javax/xml/bind/DatatypeConverter"`  

해당 메시지는 jwt 토큰을 사용할때 발생했으며, 이전 path 미션을 진행할때 비슷한 에러를 본 기억이 있다.  
당시에는 java 버전을 1.8로 명시해주고, jdk도 intellij에서 `1.8` 로 잡아주어서 해결한적이 있다.  

지금 배포환경의 ec2의 경우 jdk가 11버전으로 깔려있어서, 발생한 에러이다.  

[링크](https://stackoverflow.com/questions/43574426/how-to-resolve-java-lang-noclassdeffounderror-javax-xml-bind-jaxbexception) 에 따르면, JAXB API는 java 11부터 제거된것을 확인할 수 있다.

```
implementation group: 'javax.xml.bind', name: 'jaxb-api', version: '2.3.1'
```

의존성을 추가해주어서 해결했다.  

---


# DataBase

## ON DELETE CASCADE
- 외래키 걸어둔 자식 테이블의 데이터도 같이 지워지게 한다.

```sql
create table if not exists LINE
(
    id bigint auto_increment not null,
    primary key(id)
);

create table if not exists SECTION
(
    id bigint auto_increment not null,
    line_id bigint not null,
    primary key(id),
    foreign key (line_id) references line(id) on delete cascade
);
```

LINE이 삭제되는경우, 삭제되는 LINE의 ID를 로 가진 SECTION들도 삭제되기를 기대한다.  
도메인쪽에서 이런작업을 일일히 해줄 필요가 없고, `on delete cascade` 로 손쉽게 처리가 가능하다.  
대신 또 연쇄적으로 테이블이 지워질 수 있는 부분들이 있을수도 있으므로, 사용하기전에 외래키간의 관계를 잘 확인해보아야 한다.  

---