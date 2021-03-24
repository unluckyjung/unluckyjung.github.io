---
title: Java Collections Frequency 사용법
date: 2021-03-24-23:00
categories:
- Java

tags:
- Java
- JDK
- Collections

---

## Collections Frequency 사용법
> Collections Frequency 사용법을 알아봅니다.

### Goal
- Collecions.frequency 사용법에 대해서 알아본다.

---
  
## 기능
> Collection 안에 있는 객체가 몇번 등장했는지를 리턴해준다.

```java
Collections.frequency(객체를 담고 있는 컬렉션 인스턴스, 기대하는 객체)
```

- return 값은 일치하는 개수가 된다.

## 예제 코드

```java
import java.util.Arrays;
import java.util.Collections;
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 1, 1, 2);
        System.out.println(Collections.frequency(numbers, 1));
    }
}
```

- number안에 1이 몇개나 있는지를 확인해본다.

```
// result

3
```

<br>

```java
import java.util.Arrays;
import java.util.Collections;
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 1, 1, 2);
        int n = 1;
        System.out.println(getCountUseForLoop(numbers, n));
        System.out.println(getCountUseStream(numbers, n));

        System.out.println(Collections.frequency(numbers, n));
    }

    private static int getCountUseForLoop(final List<Integer> numbers, final int n,) {
        int count = 0;
        for (int num : numbers) {
            if (num == n) {
                count++;
            }
        }
        return count;
    }

    private static int getCountUseStream(final List<Integer> numbers, final int n) {
        return (int) numbers.stream()
                .filter(num -> num == n)
                .count();
    }
}
```

- for loop나 stream을 사용하는것보다 훨씬 깔끔하게 구현이 되는것을 볼 수 있다.


### Map을 사용하는 경우

```java
import java.util.Collections;
import java.util.HashMap;
import java.util.Map;

public class Main {
    public static void main(String[] args) {
        Map<String, Integer> score = new HashMap<>();
        score.put("JYS", 100);
        score.put("Fortune", 100);

        System.out.print("100점인 유저는 ");
        System.out.print(Collections.frequency(score.values(), 100) + "명 있습니다.");

        System.out.println("이름이 JYS인 유저는 ");
        System.out.print(Collections.frequency(score.keySet(), "JYS") + "명 있습니다.");
    }
}
```

```
// result

100점인 유저는 2명 있습니다.
이름이 JYS인 유저는 1명 있습니다.
```

- 매칭되는 경우의 수를 아주 간결하게 찾을 수 있게된다.
- key는 중복되지 않기 때문에 이름에 대한 검색은 올바르지 못한 예제이지만, `그냥 이렇게 사용할 수 있다~` 라고만 보고 가면 되겠습니다.

---

## Reference
- https://docs.oracle.com/javase/7/docs/api/java/util/Collections.html#frequency(java.util.Collection,%20java.lang.Object)
- https://www.geeksforgeeks.org/java-util-collections-frequency-java/
