---
title: Java Arrays.asList vs new ArrayList
date: 2021-03-07-18:00
categories:
- Java

tags:
- Java
- List
- Array

---

## Arrays.asList와 new ArrayList<>(Arrays.asList) 의 차이
> 읽기용, 일반적으로 알고 있는 ArrayList

- **Goal**
- `Arrays.asList`와 `new ArrayList<>(Arrays.asList)` 의 차이를 알아본다. 
  
---

## asList로 만든 인스턴스
> add가 가능할까?

```java
public class Main {
    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1, 2, 3);
        list.add(4);
    }
}
```

- 다음과 같은 코드가 있다고 해보자.
- 해당 파일은 정상적으로 컴파일이 될것 처럼 보인다.

![image](https://user-images.githubusercontent.com/43930419/110234681-2abc1a00-7f6f-11eb-956c-28c75e9d27b0.png)

![image](https://user-images.githubusercontent.com/43930419/110233643-92229b80-7f68-11eb-8e44-b4b40a92d0a9.png)

- 에러가 표시되지도 않고, 심지어 우리의 똑똑한 인텔리제이는 add가 가능하다고도 알려주고 있다.

<br>

![image](https://user-images.githubusercontent.com/43930419/110233660-a8305c00-7f68-11eb-8f91-608693dd7172.png)

- 하지만, 컴파일을 하면 다음과 같이 에러가 발생한다.
- 에러메시지만 봐서는 도대체 왜 에러가 발생한지를 모르겠다.

---

## asList를 통해 반환된것은 읽기만 가능하다.
> 값의 복사도 이루어지지 않고, 고정된 길이의 List가 리턴된다.

![image](https://user-images.githubusercontent.com/43930419/110233583-2b04e700-7f68-11eb-90d0-c791c6abb7d5.png)

- 오라클 [공식 문서](https://docs.oracle.com/javase/8/docs/api/java/util/Arrays.html#asList-T...-)에보면 다음과 같이 써있다. (이미지를 클릭하면 커집니다)
  - `Changes to the returned list "write through" to the array.`
  - 고정된 길이로 리턴하고, 읽기만 가능하다고 써있다.

- 반환형도 `java.util.ArrayList`가 아닌, `java.util.Arrays.ArrayList` 이다.

```java
public class Main {
    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1, 2, 3);
        list.contains(1);
        list.get(0);
        list.size();
    }
}
```

- 다음과 같이 읽기만 하는 행위에 대해서는 정상적으로 수행한다.

```java
public class Main {
    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1, 2, 3);
        list.set(0, 10);
        System.out.println(list.toString());
    }
}
```

```
> Task :Main.main()
[10, 2, 3]
```

- 하지만 다음과 같은 작업은 가능하다.
- [asList 구현](http://hg.openjdk.java.net/jdk8/jdk8/jdk/file/687fd7c7986d/src/share/classes/java/util/Arrays.java#l3800) 을 보면, 배열의 형태로 돌려주는것을 볼 수 있다.
- 쉽게 생각하면 `Array.asList`로 만들어진것은 하나의 고정된 길이의 배열이라고 생각하면 된다.
- 이렇게 때문에 길이를 변경시키는 add remove와 같은 작업은 불가능하나, **이미 만들어진 범위내에서는 수정이 가능하다.**
- 즉 List처럼 보이나, List인것처럼 보이게 배열을 포장한 형태라고 이해하면 된다. 

---

## new ArrayList<>(Arrays.asList())
> 모든 ArrayList의 모든 api를 사용 할 수 있다.

```java
public class Main {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3));
        list.add(4);
        list.remove(0);
        list.remove(Integer.valueOf(2));
    }
}
```

- add, remove등 모든 기능을 수행할 수 있다.

---


## Reference
- https://www.programcreek.com/2014/05/top-10-mistakes-java-developers-make/
- https://docs.oracle.com/javase/8/docs/api/java/util/Arrays.html#asList-T...-
- https://stackoverflow.com/questions/25729553/arraylist-vs-the-list-returned-by-arrays-aslist
- https://stackoverflow.com/questions/25447502/regarding-immutable-list-created-by-arrays-aslist