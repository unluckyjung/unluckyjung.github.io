---
title: Java static 메소드를 mocking 하여 테스트하기
date: 2021-12-20-16:00
categories:
- TestCode

tags:
- TestCode
- Junit5
- Mock
- Mockito

---

## static 메소드를 Mockito를 이용해 mocking 해봅니다.
> `MockedStatic<T>`

---

## Goal
- static 메소드를 mocking 하는법을 알아봅니다.
- 다른 클래스에서도 mocking이 유지되는지를 확인해봅니다.

---

## 개요
- 우아한테크코스 근로의 미션 제작팀에서 채점 자동화 방안을 찾던중, static method를 모킹해야하는일이 생겼습니다.

### 테스트에서 모킹을 하려는 메서드

```java
import java.util.Random;

public class Randoms {
    private static final Random RANDOM = new Random();

    private Randoms() {
    }

    // 범위내의 랜덤한 숫자 1개를 제공하는 메서드
    public static int nextInt(final int startInclusive, final int endInclusive) {
        // 범위에 대한 예외처리 로직이 있다고 가정
        ...

        return startInclusive + RANDOM.nextInt(endInclusive - startInclusive + 1);
    }
}
```

- 랜덤한 값을 뽑아내는 static method인 `nextInt()` 에 대한 모킹이 필요해졌습니다.

### 모킹을 실패하는 예시

```java
import static org.assertj.core.api.Assertions.assertThat;
import static org.mockito.BDDMockito.given;

import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

@ExtendWith(MockitoExtension.class)
class RandomsTest {

    @Mock
    private Randoms randoms;

    @Test
    @DisplayName("원하는 Random값(5)이 나오면, 테스트를 통과한다.")
    void randomSetsTest() {
        given(Randoms.nextInt(1, 2)).willReturn(5);
        assertThat(Randoms.nextInt(1, 2)).isEqualTo(5);
    }
}
```

- static 메소드를 일반 메소드를 Mocking 하듯이 했더니, 아래와 같은 메시지를 띄우며 실패했습니다.

```
org.mockito.exceptions.misusing.MissingMethodInvocationException: 
when() requires an argument which has to be 'a method call on a mock'.
For example:
    when(mock.getArticles()).thenReturn(articles);

Also, this error might show up because:
1. you stub either of: final/private/equals()/hashCode() methods.
   Those methods *cannot* be stubbed/verified.
   Mocking methods declared on non-public parent classes is not supported.
2. inside when() you don't call method on mock but on some other object.
```
 
---

## 해결방법

### mockStatic을 이용합니다.

- mockito 버전 `3.4.0` 이상부터 제공하는 기능입니다.
- 일단 먼저 `3.4.0` 버전이상으로 의존성을 추가해줍니다.

```java
testImplementation 'org.mockito:mockito-inline:3.6.0'
```

> 모킹할 클래스를 가지고 MockStatic 객체를 선언해줍니다.

```java
private static MockedStatic<모킹할 메소드를 가진 클래스> instance;
// 예시
private static MockedStatic<Randoms> randoms;
```

```java
@BeforeAll
public static void beforeALl() {
    instance = mockStatic(모킹할 메소드를 가진 클래스);
    // 예시
    randoms = mockStatic(Randoms.class);
}
```

```java
private static MockedStatic<Randoms> Randoms;

@BeforeAll
public static void beforeALl() {
    randoms = mockStatic(Randoms.class);
}

@AfterAll
public static void afterAll() {
    randoms.close();
}
```

> 주의 : `randoms.close()`를 반드시 테스트 수행 후 호출해야합니다.  

- 한 스레드에서 staticMocking 등록은 한번밖에 못하므로, 생략할경우 아래와 같은 에러메시지를 만나게 됩니다.
  
```
org.mockito.exceptions.base.MockitoException: 
For utils.Randoms, static mocking is already registered in the current thread

To create a new mock, the existing static mock registration must be deregistered
```


### 모킹

```java
given(모킹할 메소드를 가진 클래스.메소드명()).willReturn(모킹할 값);

// 예시
given(Randoms.nextInt(1, 2)).willReturn(5);
```

- 메소드를 모킹해줍니다.

---


## 전체 코드
> mockStatic을 static이나 멤버 변수가아닌 로컬 변수로 두어도 문제없이 작동합니다.

---

```java
import static org.assertj.core.api.Assertions.assertThat;
import static org.mockito.BDDMockito.given;
import static org.mockito.Mockito.mockStatic;

import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.ValueSource;
import org.mockito.MockedStatic;

class RandomsTest {

    @ParameterizedTest
    @ValueSource(ints = {-1, 0, 1})
    @DisplayName("원하는 Random값이 나오면, 테스트를 통과한다.")
    void randomSetsTest(final int expectValue) {
        MockedStatic<Randoms> randoms = mockStatic(Randoms.class);

        given(Randoms.nextInt(1, 2)).willReturn(expectValue);

        assertThat(Randoms.nextInt(1, 2)).isEqualTo(expectValue);

        randoms.close();
    }
}
```

- 1,2 범위에서 나올 수 없는 `-1, 0` 의 경우에도 테스트를 통과하는것을 확인할 수 있습니다.

---

## Try-with-resource를 사용하기
> 자원할당과 해제를 자동으로

### 테스트할 도메인  

```java
public class BlogerName {

    private BlogerName() {
    }

    public static String getName() {
        return "unluckyJung";
    }
}
```

- `getName()` 을 할시 항상 **"unluckyjung"**을 반환하는 메서드입니다.
- 이 메서드를 호출할경우 unluckyjung이 아닌 **JungYoonsung**을 반환하도록 수정해보겠습니다.


### 테스트 코드

```java
import domain.BlogerName;

public class OtherClass {

    public static String getUserName() {
        System.out.println(BlogerName.getName());
        return BlogerName.getName();
    }
}
```


```java
import static org.assertj.core.api.Assertions.assertThat;
import static org.mockito.BDDMockito.given;
import static org.mockito.Mockito.mockStatic;

import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.mockito.MockedStatic;
import other.OtherClass;

class BlogerNameTest {

    @DisplayName("unluckyJung이 아니라, JungYoonsung이 나오면, 테스트를 통과한다.")
    @Test
    void blogerNameTest() {
        // mocking 전
        assertThat(OtherClass.getUserName()).isEqualTo("unluckyJung");

        try (MockedStatic<BlogerName> blogerName = mockStatic(BlogerName.class)) {
            given(BlogerName.getName()).willReturn("JungYoonsung");
            assertThat(OtherClass.getUserName()).isEqualTo("JungYoonsung");
        }
    }
}
```

- 사용하고 있는 자바의 버전이 7 이상인경우, 가능하면 `try-with-resource`를 사용하는것이 `close()`를 호출하지 않아 발생하는 문제를 최소화 할 수 있어서 추천 합니다. 
- 장점에 대한 좀더 자세한 내용은 [링크](https://www.baeldung.com/mockito-mock-static-methods#mocking-a-no-argument-static-method)를 참조하시는걸 추천드립니다.

![image](https://user-images.githubusercontent.com/43930419/146736717-24e30260-77e8-438b-bac3-79732e4e194c.png)

- 테스트코드가 아닌 다른 클래스에서 `getName()` 호출하는 경우에도, Mocking이 유지되는것을 확인할 수 있습니다. (이미지 클릭시 커집니다.)

---

## Conclusion
- mockito 3.4.0 버전 이상에서 제공하는 mockStatic을 이용해 정적 메서드를 모킹할 수 있다.
- 자원 할당 해제가 굉장히 중요하다. 가능하면 try-with-resource를 이용하자.

---

## Github
- 해당코드들은 전부 [github](https://github.com/unluckyjung/blog-codes/tree/main/static-method-mocking)에서 확인 하실 수 있습니다.

---

## Reference
- https://javadoc.io/static/org.mockito/mockito-core/4.2.0/org/mockito/Mockito.html#static_mocks
- https://www.baeldung.com/mockito-mock-static-methods
- https://www.baeldung.com/bdd-mockito
- https://frontbackend.com/java/how-to-mock-static-methods-with-mockito
- https://www.crocus.co.kr/1705
- https://asolntsev.github.io/en/2020/07/11/mockito-static-methods/