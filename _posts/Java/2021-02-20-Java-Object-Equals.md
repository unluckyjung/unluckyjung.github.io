---
title: Java 객체끼리 같은지 비교하기
date: 2021-02-20-17:00
categories:
- Java

tags:
- Java

---

## 객체끼리 같은지를 비교해봅니다.
> equals와 hashCode를 오버라이딩 해준다.

---

## 코드1

```java
public class Name {
    private final String name;

    public Name(final String name) {
        this.name = name;
    }
}
```

- 다음과 같은 Name 클래스를 정의했다고 해보자.


## 테스트 코드

```java
package racingcar.domain;

import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

import static org.assertj.core.api.AssertionsForClassTypes.assertThat;

public class NameTest {
    @Test
    @DisplayName("이름이 같은 경우")
    void nameIsEquals() {
        Name name = new Name("fortune");
        assertThat(name).isEqualTo(new Name("fortune"));
    }
}
```

- 해당 테스트 코드를 실행시키면, `실패`한다
- 왜냐하면 두개의 객체는 다른 해시값을 지니기 때문!
- 그러면 어떻게 해야할까?

---

## 코드2

```java
package racingcar.domain;

import java.util.Objects;

public class Name {
    private final String name;

    public Name(final String name) {
        this.name = name;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Name name1 = (Name) o;
        return Objects.equals(name, name1.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name);
    }
}
```

- `equals` 와 `hashCode`를 오버라이딩 해서 구현해준다.
- 이제 테스트 코드에서는 true 값을 받게 된다.

---

## name을 String이 아닌 굳이 Name으로 포장하는 이유
> 신뢰성 있는 정보를 쓰기 위해서

- String 같은 값으로 구현한다면, 해당 name이 올바른것인지를 검증하는 절차가 계속 필요하다.
  - 예를들어 name이 영어로만 이루어져있기를 기대하는데, 숫자나 특수문자가 들어간다면? 
- String이 아닌 Name에 담아 관리한다면, 생성자 단계에서 올바른 이름인지를 확인하는 과정을 가질 수 있다.
  - 이로 인해 `Name` 객체를 쓰는 다른 도메인에서는 이름이 보장된 값임이 이미 보증되었으므로, 검증로직을 생략하고 맘편히 사용할 수 있게된다.