---
title: JAVA 정규식으로 숫자 추출하기
date: 2021-02-12-21:00
categories:
- Java

tags:
- Java
- Regex

---

## 자바에서 정규식을 이용해 숫자를 추출해 봅니다.
> 들어온것이 숫자로만 이루어져 있는지도 확인해 봅니다.

---

## 정규식 패턴을 한번만 만들기

```java
    private static final Pattern GET_NUMBER = Pattern.compile("[0-9]+");
    private static final Pattern IS_ONLY_NUMBER = Pattern.compile("^[0-9]*?");
```

- Pattern을 계속 만들어 내는것은 자원낭비가 심하다.
- 그러므로, 해당 부분을 `static` 으로 한번만 만들게 해놓고 재활용해 사용한다.

---

## 모두가 숫자인지 확인하기

```java
    public boolean isOnlyNumber(final String str) {
        return IS_ONLY_NUMBER.matcher(str).matches();
    }
```

- `matcher.matches()` 를 통해 **전부**가 일치되는지 확인한다.

---

## 숫자만 추출해내기

```java
    public String getOnlyNumber(final String str) {
        StringBuilder sb = new StringBuilder();
        Matcher matcher = GET_NUMBER.matcher(str);

        while (matcher.find()) {
            sb.append(matcher.group());
        }
        return sb.toString();
    }
```

- 매칭되는 것을 찾는 matcher를 만들어두고, 매칭되는게 있다면 `gruop()`을 통해 매칭되는 문자열을 뽑아낸다.

```java
        while (IS_ONLY_NUMBER.matcher(str).find()) {
            sb.append(IS_ONLY_NUMBER.matcher(str).group());
        }
```

- 해당 방법으로는 불가능하다.
- 일치되는 경우가 존재하는경우, `find()` 는 계속 같은 위치만 보게 되기 때문이다.
  - `find()`는 찾은곳의 위치로 이동한다. 
  - 이후 `find()`를 호출한다면 해당 위치부터 정규식에 매칭되는것이 있는지 찾아나간다.

---

## 전체 소스

```java
package regex;

import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class Regex {
    private static final Pattern GET_NUMBER = Pattern.compile("[0-9]+");
    private static final Pattern IS_ONLY_NUMBER = Pattern.compile("^[0-9]*?");

    public boolean isOnlyNumber(final String str) {
        return IS_ONLY_NUMBER.matcher(str).matches();
    }

    public String getOnlyNumber(final String str) {
        StringBuilder sb = new StringBuilder();
        Matcher matcher = GET_NUMBER.matcher(str);

        while (matcher.find()) {
            sb.append(matcher.group());
        }
        return sb.toString();
    }
}

```

---


## 테스트에 사용한 테스트 코드

```java
package regex;

import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

import static org.assertj.core.api.AssertionsForClassTypes.assertThat;


public class RegexTest {
    Regex regex = new Regex();

    @Test
    @DisplayName("123456가 숫자로만 이루어졌는지 확인하기")
    void checkIsOnlyNumbers() {
        String str = "123456";
        assertThat(regex.isOnlyNumber(str)).isEqualTo(true);
    }

    @Test
    @DisplayName("123-456-789 가 숫자로만 이루어졌는지 확인하기")
    void checkPhoneNumberContainsOnlyNumbers() {
        String str = "123-456-789";
        boolean result = regex.isOnlyNumber(str);
        assertThat(result).isEqualTo(false);
    }

    @Test
    @DisplayName("123-456-789 에서 숫자만 가져오기")
    void checkNumbers() {
        String str = "123-456-789";
        String result = regex.getOnlyNumber(str);
        assertThat(result).isEqualTo("123456789");
    }
}
```

---

## Reference
- https://coding-factory.tistory.com/529