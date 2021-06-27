---
title: 테스트 메소드별로 독립된 환경에서 테스트하기
date: 2021-05-08-16:00
categories:
- TestCode

tags:
- TDD
- TestCode
- Spring

---

## 스프링 테스트 코드를 작성할때 테스트 순서에 영향받지 않고, 독립적으로 테스트 해봅니다.
> Spring 5, H2 DB 환경

---

## Goal

- `@Transactional` 어노테이션을 사용해봅니다.
- `@Transactional` 어노테이션이 `auto_increment` 를 커버하지 못하는것을 확인해봅니다.
- `@Sql` 어노테이션을 사용해봅니다.

---

## @Transactional

- 어노테이션을 달아주면 [TRUNCATE](https://wikidocs.net/4019) 처럼 모든 Row를 삭제 시켜준다.
- 어노테이션의 위치에 따라 동작이 살짝 다르다.
  - 클래스 위에 붙여주면, `@BeforeEach` 처럼 모든 테스트 메소드를 수행하기전에 삭제시켜준다.
  - 메소드 위에 붙여주면, 해당 메소드를 수행하기 전에 삭제 시켜준다.


### @Transactional 롤백 범위
- 테이블에 대한 롤백은 시켜주나, `auto-Increment`에 대한 롤백은 되지 않는다.
- 즉, `auto-Increment`에서 독립적으로 테스트를 하기 원한다면, auto-Intcrement에 대한 시작점을 다시 세팅해주어야 한다.
  - 방법은 [ALTER](https://amaze9001.tistory.com/28) 를 이용하는 방법이 있다.

- 에시를 조금 더 상세히 들자면


```sql
create table STATION
(
    id bigint auto_increment not null,
    name varchar(255) not null unique,
    primary key(id)
);
```

- 같은 테이블이 있다고 해보자.
- **A라는 테스트**는 삽입 후 삭제가 잘 되는지를 확인한다.
- **B라는 테스트**는 삽입 후, 삽입할때 넣어준 name을 가진 STATION의 id가 기대하는 값인지를 확인한다.

- 테이블이 비어있는 상태라고 가정 해보자.
- A 테스트을 먼저 수행한다고 해보자.
  - **auto_increment**를 통해 id를 **1**로 배정받고, 삭제한다.
  - `@Transactional` 이 테이블의 row를 다 삭제해준다.
  - 이때 **auto_increment**는 초기화가 되지 않는다.

- 문제는 B 테스트를 수행할 때이다.
  - A 테스트 **이전**에 수행 된다면, 삽입 후 기대되는 id는 1이 될것이다.
  - A 테스트 **이후**에 수행 된다면, 삽입 후 기대하는 id는 2가 될것이다.
- 다른 테스트에 영향이 받는 상황이 되어버린다.

- 테이블에 `TRUNCATE` 를 사용해주거나, 클래스나 메소드에 `@Transactional` 을 붙여주어도 **Auto-Increment는 초기화 되지 않는다.**
- ALTER를 이용해 다시 재설정하거나, 그냥 테이블을 날려버리는수밖에 없다.

```sql
ALTER TABLE station AUTO_INCREMENT = 1;

-- 혹은

DROP TABLE STATION;

create table if not exists STATION
(
    id bigint auto_increment not null,
    name varchar(255) not null unique,
    primary key(id)
);
```

- 와 같은 쿼리를 수행시켜주어야 한다.
- 그렇다면 매번 모든 테스트 클래스의 `@BeforeEach` 에 이런 쿼리를 넣어주어야 할까?
- 그 불편함을 `@SQL` 어노테이션으로 해소할 수 있다.

---

### @SQL

- 테스트를 실행하기전에 sql 스크립트를 실행하게 할 수 있다.
- 롤백 기능으로도 훌륭하나, 아예 테이블 자체를 지웠다가 만드는 작업 + Insert를 수행하고 싶을떄 용이하게 사용할 수 있다.
- 여러 스크립트를 돌리는것도 가능하다 
- **클래스단**에 달아줄 경우, 모든 메소드를 수행하기전에, 매번 스크립트가 실행된다.
- **메소드단**에 달아줄 경우, 해당 메소드를 수행하기전에, 스크립트가 실행된다.


```sql
create table if not exists LINE
(
    id bigint auto_increment not null,
    name varchar(255) not null unique,
    color varchar(20) not null,
    primary key(id)
);
```

```java
@JdbcTest
@Sql("classpath:tableInit.sql")
class LineRepositoryTest {

...
    @BeforeEach
    void setUp() {
        lineRepository = new LineRepository(jdbcTemplate);
        String query = "INSERT INTO line(color, name) VALUES(?, ?)";
        jdbcTemplate.update(query, "bg-red-600", "신분당선");
        jdbcTemplate.update(query, "bg-green-600", "2호선");
    }

    //@Test
    // void 데이터 삽입후 삽입 확인 테스트 {
    //     ...
    // }

    @DisplayName("이름이랑 색깔을 입력받으면, DB에 Line을 생성하고, id를 반환한다.")
    @Test
    void save() {
        Line line = new Line("bg-blue-600", "1호선");
        assertThat(lineRepository.save(line).getId()).isEqualTo(3L);
    }
}
```

- 만약 `데이터 삽입후, 삽입 확인 테스트 ` 라는 테스트가 있었으면, 
  - 해당 테스트는 `save()` 테스트에 영향을 줄것이다.
  - `@JdbcTest` 안의 `@Transactional` 어노테이션이 Row 자체를 삭제 해줄것이나, `auto-increment` 를 초기화 해주진 못한다. 
  - **기대하는 id값이 4L이 될 수도 있기 떄문이다.**
- `save()` 테스트는 **순서에 따라서 성공, 실패 여부가 갈리는 테스트가 될것이다.**

- 이때 테스트를 순서를 보장하려고 하기보다는, 다른 테스트에 독립된 테스트를 구성해보는게 어떨까?

```sql
-- tableInit.sql

DROP TABLE LINE;

create table if not exists LINE
(
    id bigint auto_increment not null,
    name varchar(255) not null unique,
    color varchar(20) not null,
    primary key(id)
);
```

- `@Sql` 를 사용한다면, 클래스별로 중복된 코드없이 손쉽게 테이블 자체를 DROP, CREATE 할 수 있다.
- 물론 ALTER를 이용해 auto_increment만 초기화 해주는것도 가능하다.


---

## DROP and CREATE vs ALTER
> 그렇다면 auto_increment를 초기화 하기 위해 어떤 방법을 사용하면 좋을까?

![image](https://user-images.githubusercontent.com/43930419/117531862-a98d0c00-b01f-11eb-9edd-46bbff338072.png)

- 이부분에 대해서 리뷰어분에게 여쭈어봤고, 위와 같은 답변을 받았다.
- 일단 나의 경우에는 `@SQL` 어노테이션을 이용해 테이블 자체를 밀어버리는 방법을 사용할것 같다!
- 데이터의 양이 많다면 물론 부담이 되겠지만, 테스트용으로 가볍게 더미 데이터를 넣었다 뺏다 하는 수준이므로 괜찮지 않을까? 라는 생각을 해본다.

---

## Reference

- https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/context/jdbc/Sql.html
- https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/Transactional.html
- https://www.baeldung.com/spring-boot-data-sql-and-schema-sql
- https://javacan.tistory.com/entry/spring41-AtSql-annotation-intro
- https://wikidocs.net/4019
- https://amaze9001.tistory.com/28