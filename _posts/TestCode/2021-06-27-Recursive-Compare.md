---
title: 동일성이 아닌 동등성을 라이브러리를 이용해 비교하기
date: 2021-06-27-20:00
categories:
- TestCode

tags:
- TDD
- TestCode
- AssertJ

---

## 동등성을 비교하여 올바른 DTO가 반환되었는지 확인 해보자
> DTO의 경우 VO와 다르게 동등성을 비교하는 메소드를 오버라이딩 하지 않습니다.

- Junit5
- AssertJ

```
dependencies {
    testImplementation 'org.junit.jupiter:junit-jupiter:5.7.1'
    testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine'
    testImplementation 'org.assertj:assertj-core:3.19.0'
}
```

---

## Goal
- 동일성이 아닌 동등성을 비교하는 테스트 코드를 작성해 본다.

---

## 동일성 vs 동등성
> 완전히 같으냐?, 값만 같으냐?

### 동일성
- **동일성**의 경우에는 완전히 같은 객체인지를 확인 한다.
- 돈을 예시로 들때 1만원짜리 지폐가 있다면, 1만원 값이 같고 **일련번호**까지 완전히 같은 지폐가 동일한 지폐로 판단한다.
- **동일성**을 비교하는 방법은 일반적으로 java 기준 `equals`, `hashCode`를 오버라이딩 하여 구현한다.
  - [참고할만한 연관된 글](https://unluckyjung.github.io/java/2021/02/20/Java-Object-Equals/)

### 동등성
- **동등성**의 경우에는 보유하고 있는 값이 같은지를 확인 한다.
- 1만원짜리 지폐가 있다면, 일련번호가 달라도 같은 1만원이라면 **같은 지폐**로 판단한다.
  - 동일성이 만족되었다면, **동등성**역시 같이 만족되었다고 볼 수 있다.

---

## usingRecursiveComparison()
> AssertJ

- VO가 아닌것들을 대상으로 테스트 코드를 짤때, 동일성 비교가 아닌 동등성을 비교하고 싶다.
- `usingRecursiveComparison()` 와 같은 아주 좋은 메소드가 있다.
  - 내부 값만 같으면 테스트를 통과한다!
  - DTO와 같은 것을 비교할때 아주 용이하다.

- 내부적으로 모든값을 재귀적으로 돌면서 확인해준다.

---

### 코드1

```java

@DisplayName("UserDTO가 같으면, 통과한다.")
@Test
void test2() {
    UserDTO userDTO = new UserDTO(1, 29, "JYS");
    UserDTO expectedUserDTO = new UserDTO(1, 29, "JYS");

//        assertThat(userDTO).isEqualTo(expectedUserDTO); // fail
    assertThat(userDTO).usingRecursiveComparison().isEqualTo(expectedUserDTO);
}
```

```java
@DisplayName("유저 정보를 요청하면, 유저 DTO를 받는다.")
@Test
void test3() {
    UserService userService = new UserService();
    UserDTO expectedUserDTO = new UserDTO(1, 29, "JYS");

    assertThat(userService.getUserById(1)).usingRecursiveComparison().isEqualTo(expectedUserDTO);
}

class UserService {
    private final Map<Integer, UserDTO> userDB = new HashMap<>();

    {
        userDB.put(1, new UserDTO(1, 29, "JYS"));
    }

    public UserDTO getUserById(final int id) {
        return userDB.get(id);
    }
}
```

### 코드2

```java
@DisplayName("value 값이 전부 같으면 통과한다.")
@Test
void test4() {
    Depth1 depth1 = new Depth1(new Depth2(1), 11);

    assertThat(depth1).usingRecursiveComparison().isEqualTo(new Depth1(new Depth2(1), 11));

    assertThat(depth1).usingRecursiveComparison().isNotEqualTo(new Depth1(new Depth2(1), 22));
    assertThat(depth1).usingRecursiveComparison().isNotEqualTo(new Depth1(new Depth2(2), 11));
}

class Depth1 {
    private final Depth2 depth2;
    private final int value;

    Depth1(final Depth2 depth2, final int value) {
        this.depth2 = depth2;
        this.value = value;
    }
}

class Depth2 {
    private final int value;

    Depth2(final int value) {
        this.value = value;
    }
}
```

---

## usingRecursiveFieldByFieldElementComparator
- 응용하면 아래와 같이 작성할 수 있다.
- `usingRecursiveFieldByFieldElementComparator()` 와 `contains`를 사용해
- **순서와 상관없이** 안의 값중 동등한것이 한개라도 있는지 확인할 수 있다.

### 코드
```java
@DisplayName("전체중 일부 하나라도 동등하면, 테스트가 통과한다.")
@Test
void test() {
    List<UserDTO> users = Arrays.asList(
            new UserDTO(1, 29, "JYS"),
            new UserDTO(2, 30, "Fortune")
    );

    UserDTO expectedUserDTO = new UserDTO(1, 29, "JYS");

//        assertThat(users).contains(ExpectedUserDTO); // fail
    assertThat(users).usingRecursiveFieldByFieldElementComparator().contains(expectedUserDTO);
}
```

---


## Recursive 특정 필드값 무시
- 동등성을 비교할때, 특정 필드값은 제외하면서 비교, 확인할 수 있다.

```java
{
    id : 1
    age : 29
    name : "good"
}
```

과 같은 형태에서 몇개의 필드들은 무시하며 동등성을 확인하고 싶으면 

```java

// id를 무시하는경우
assertThat(ExpectedSomething).usingRecursiveComparsion().ignoringFields("id").EqaulsTo(something);

// id와 age 둘다 무시하는경우
assertThat(ExpectedSomething).usingRecursiveComparsion().ignoringFields("id", "age").EqaulsTo(something);
```

와 같은 형태로 특정 필드들을 무시하며 동등성을 비교할 수 있다.


---



## Reference
- https://assertj.github.io/doc/#assertj-core-recursive-comparison
- https://stackoverflow.com/questions/43316133/how-to-compare-recursively-ignoring-given-fields-using-assertj
- https://www.podo-dev.com/blogs/232