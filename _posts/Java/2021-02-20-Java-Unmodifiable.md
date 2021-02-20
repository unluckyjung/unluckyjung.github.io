---
title: Java 컬렉션을 수정불가능 하게 하기
date: 2021-02-20-17:30
categories:
- Java

tags:
- Java

---

## 수정 불가능한 컬렉션을 만들어 봅니다.
> final 만으로는 부족하다.

---

## 그냥 final만 붙이는 경우.

```java
public class FinalCollection {
    private static final List<Integer> list = new ArrayList<>();
    private static final List<Integer> constList = Collections.unmodifiableList(Arrays.asList(100, 200, 300));
    
    public static void main(String[] args) {
        list.add(10);   // 작동한다!
        System.out.println(constList.get(0)); // 100
        // list = new ArrayList<>();   // 이건 컴파일단계에서 불가능하다고 알려준다.
    }
}
```

- `list`를 보면 static final을 붙여줬지만, 수정이 가능하다!
- 왜그럴까?
    - final은 정확히는 **한번만 선언될 수 있음**을 나타내는 키워드이다.
    - 단순히 변수일때는 상관이 없지만, 객체인 컬렉션인 경우에는 선언은 됐지만 **변경될 수 있다.**
        - 객체 변수에 final을 건것이므로, 새롭게 다른 참조값을 지정할 수는 없다.
        - 하지만 객체 자체는 `immutable` 되지 않는다. 즉 속성의 경우에는 변경이 가능한 상태가 유지 된다.

---

## unmodifiable을 붙이는 경우

```java
public class FinalCollection {
    private static final List<Integer> constList = Collections.unmodifiableList(Arrays.asList(100, 200, 300));
    
    public static void main(String[] args) {
        constList.add(0, 1000); // 예외 발생!
    }
}
```

![image](https://user-images.githubusercontent.com/43930419/108589767-64075e00-73a3-11eb-9a94-43981b57a822.png)

-  `Collections.unmodifiableList` 을 넣어준다면, 수정이 불가능 해진다.
-  즉 컬렉션이 수정되는것을 원치 않는다면, `unmodifiable` 을 이용하자.

---

## Reference
- [final](https://docs.oracle.com/javase/tutorial/java/IandI/final.html)
- [collections](https://docs.oracle.com/javase/7/docs/api/java/util/Collections.html)