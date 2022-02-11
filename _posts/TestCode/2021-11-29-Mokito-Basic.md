---
title: Mockito 기초 사용법
date: 2021-11-29-18:46
categories:
- TestCode

tags:
- TestCode
- Junit5
- Mock
- Mockito

---

# Mockito 사용법
> Mockito 의존성 설정, 기초적인 사용법을 알아봅니다.

---

## Goal
- Mokcito의 설정, 기본적인 사용법을 알아봅니다.
- `@ExtendWith(MockitoExtension.class)` 사용시 필요한 의존성을 알아봅니다.

---

## Mockito 기본 사용법

### 의존성 추가
> gradle 기준

```java
    testImplementation 'org.mockito:mockito-core:3.11.2'
    testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine:5.7.0'
```

>  전체 의존성  


```java
plugins {
    id 'java'
}

group 'org.example'
version '1.0-SNAPSHOT'

repositories {
    mavenCentral()
}

dependencies {
    testImplementation 'org.junit.jupiter:junit-jupiter-api:5.7.0'
    testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine:5.7.0'
    testImplementation 'org.assertj:assertj-core:3.11.1'

    // mockito 의존성
    testImplementation 'org.mockito:mockito-core:3.11.2'
}

test {
    useJUnitPlatform()
}
```

### Mocking할 객체
```java
public class Member {

    private final int id = 7;

    public int getId() {
        return id;
    }
}
```

- 현재 member 객체는 항상 `getId()`시 항상 7을 반환하고 있습니다.

---

### 테스트 코드
> 통과

```java

// 어노테이션 기반으로 member에 대해서 모킹을 시도합니다.
@Mock
private Member member;

// 어노테이션 기반으로 모킹을 했음을 명시적으로 알립니다.
@BeforeEach
void setUp() {
    // 과거에는 initMocks(this) 라는 메소드 명이었지만, 버전업이 됨에 따라 openMocks로 메소드명이 바뀌었습니다.
    MockitoAnnotations.openMocks(this);
}

// member로부터 getId() 메소드를 호출하는경우 10을 리턴하게 합니다.
given(member.getId()).willReturn(10);
```


```java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;
import org.mockito.junit.jupiter.MockitoExtension;

class MemberTest {

    @Mock
    private Member member;

    @BeforeEach
    void setUp() {
        MockitoAnnotations.openMocks(this);
    }

    @Test
    void test1() {
        given(member.getId()).willReturn(10);
        assertThat(member.getId()).isEqualTo(10);
    }
}
```

- member 인스턴스를 만드는 작업 조차 없었지만, **NPE**가 발생하지 않고 7이 아닌 10을 리턴하는것을 테스트 코드를 통해 확인할 수 있습니다.

---

## @ExtendWith(MockitoExtension.class)를 이용하여, init이나 open 메소드 호출 생략하기

- https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#junit5_mockito
- https://javadoc.io/doc/org.mockito/mockito-junit-jupiter/latest/org/mockito/junit/jupiter/MockitoExtension.html
- https://stackoverflow.com/questions/40961057/how-to-use-mockito-with-junit5

### 의존성 추가

```java
{
    // testImplementation 'org.mockito:mockito-core:3.11.2' // 이전 의존성
    testImplementation 'org.mockito:mockito-junit-jupiter:3.11.2'
}
```

- `core` 를 그대로 사용하는것이 아니라, 한층 더 [확장된](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#junit5_mockito) `junit-jupiter`를 사용해야합니다.
- 참고로 스프링 부트를 사용하는경우, `spring-boot-starter-test` 에 의존성이 알아서 추가가 되어있습니다.

### 테스트 코드

```java
@ExtendWith(MockitoExtension.class) // 클래스단에 해당 어노테이션을 달아, 클래스가 Mockito를 사용함을 명시적으로 알립니다.
class MemberTest2 
```

```java
@ExtendWith(MockitoExtension.class)
class MemberTest2 {

    @Mock
    private Member member;

    @Test
    void test1() {
        given(member.getId()).willReturn(10);
        assertThat(member.getId()).isEqualTo(10);
    }
}
```

<br>

### 생략된 작업

```java
@BeforeEach
void setUp() {
    MockitoAnnotations.openMocks(this);
}
```

- `MockitoAnnotations.openMocks(this)` 작업을 생략해도 Mocking이 정상적으로 이루어지는것을 확인할 수 있습니다.

---

## Code
- 해당 코드들은 전부 [github](https://github.com/unluckyjung/blog-codes/tree/main/mockito-basic) 에서 보실 수 있습니다.

---

## Reference
- https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#junit5_mockito
- https://javadoc.io/doc/org.mockito/mockito-junit-jupiter/latest/org/mockito/junit/jupiter/MockitoExtension.html
- https://mincong.io/2020/04/19/mockito-junit5/
- https://tech.lattechiffon.com/2021/07/03/junit5%EC%99%80-mockito%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-mock-test-java/