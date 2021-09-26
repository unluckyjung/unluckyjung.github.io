---
title: (Java) Enum 캐싱
date: 2021-09-26-18:00
categories:
- Java

tags:
- Java
- JDK
- Enum

---

## Enum을 캐싱
> Enum을 캐싱하여 원하는값을 빠르게 찾아봅니다.

## Goal
- HashMap을 이용해, 자주 조회되는 상수를 빠르게 찾아봅니다.

---

## 일반적으로 쓰는 방법

```java
public enum Rank {
    RANK1(5),
    RANK2(4),
    RANK3(3),
    RANK4(2),
    RANK5(1),
    FAIL(0);

    private final int matchCount;

    Rank(final int matchCount) {
        this.matchCount = matchCount;
    }
}
```

- 다음과 같은 Rank Enum이 있다고 해봅시다.
- 여기서 **mathCount**에 따른 RANK 를 얻어오고 싶다면 어떻게 할까요?

```java
public enum Rank {
    RANK1(5),
    RANK2(4),
    RANK3(3),
    RANK4(2),
    RANK5(1),
    FAIL(0);

    private final int matchCount;

    Rank(final int matchCount) {
        this.matchCount = matchCount;
    }

    public static Rank getRankByMatchCount(final int matchCount) {
        return Arrays.stream(values())
                .filter(rank -> rank.matchCount == matchCount)
                .findFirst()
                .orElse(FAIL);
    }
}
```

- 쉽게 떠올릴 수 있는 방법은 위와 같습니다.
- 이 방법은 Enum 종류가 `N`개 있다고 했을때, 매번 `O(N)`의 복잡도의 연산을 수행하겠죠.
- 종류가 많다면, 그리고 조회가 빈번하다면 개선할 필요성이 보입니다.

---

## 캐싱을 이용한 Enum 조회
> HashMap을 이용합니다.

```java
import java.util.HashMap;
import java.util.Map;

public enum Rank {
    RANK1(5),
    RANK2(4),
    RANK3(3),
    RANK4(2),
    RANK5(1),
    FAIL(0);

    private static final Map<Integer, Rank> RANK_MAP = new HashMap<>();

    static {
        for (Rank rank : values()) {
            RANK_MAP.put(rank.matchCount, rank);
        }
    }

    private final int matchCount;

    Rank(final int matchCount) {
        this.matchCount = matchCount;
    }

    public static Rank getRankByMatchCount(final int matchCount) {
        if(!RANK_MAP.containsKey(matchCount)){
            return Rank.FAIL;
        }
        return RANK_MAP.get(matchCount);
    }
}
```

- 발상은 간단합니다. 미리 저장을 해두는거죠.
- 찾으려는 속성에 따른 Enum을 HahMap에 저장해놓고(캐싱), Enum을 찾아 반환하면 `O(N)`이었던 복잡도를 `O(1)`로 줄일 수 있게 됩니다.

---

## 참고
- Enum의 식별자를 통해 Enum을 얻어오는 [valueOf](https://docs.oracle.com/javase/8/docs/api/java/lang/Enum.html#valueOf-java.lang.Class-java.lang.String-) 의 경우에도 `O(1)`로 연산을 수행합니다.

```java
public class App {
    public static void main(String[] args) {
        Rank.valueOf("RANK1");
    }
}
```

---

## Reference
- https://www.baeldung.com/java-enum-values
- https://stackoverflow.com/questions/51631114/checking-if-a-string-is-contained-in-an-enum-set-in-o1
- https://docs.oracle.com/javase/8/docs/api/java/lang/Enum.html#valueOf-java.lang.Class-java.lang.String-
