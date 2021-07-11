---
title: Java Array to List, List to Array
date: 2021-07-11-23:00
categories:
- Java

tags:
- Java
- JDK

---

## Java Array to List, List to Array
> 배열을 리스트로, 리스트를 배열로 바꾸어 본다.

## Goal
- Array를 List로 변환 해본다.
- List를 Array로 변환 해본다.

---

## Array to List

### Integer[] (unprimitive type)

```java
private static void ArrayToListInteger() {
    Integer[] arr = {1, 2, 3, 4, 5};

    List<Integer> list = Arrays.asList(arr);
}
```

- `Arrays.asList()` 로 변환이 가능하다.

### int[] (primitive type)

```java
private static void ArrayToListInt() {
    int[] arr = {1, 2, 3, 4, 5};

    //  List<Integer> list = Arrays.asList(arr); // primitive 불가능
    List<Integer> list = Arrays.stream(arr).boxed().collect(Collectors.toList());
}
```

- 스트림을 이용하여, 한번 boxing 후 List로 반환해야 한다.

---

## List to Array

### Integer[] (unprimitive type)

```java
private static void ListToArrayInteger() {
    List<Integer> list = Arrays.asList(1, 2, 3);
    Integer[] arr = list.toArray(new Integer[0]);
}
```

- `toArray(new Integer[0])`으로 해결이 가능하다.
- 여기서 `Integer[0]` 에서 **0** 이 의미하는 바는 아래와 같다.
  - List를 toArray 를 통해 변환할때, `max(파라메터로 들어가는 배열 크기, 실제 list 크기)` 로 배열의 크기가 결정된다.
  - 즉 `Integer[0]`의 크기는 0이고, list의 크기는 3이므로, 3으로 크기가 잡힌다.

![image](https://user-images.githubusercontent.com/43930419/124480804-9cec2080-dde2-11eb-973b-f13b8a406b7d.png)

- 위와 같이 사이즈를 5로 준경우에는, array의 크기는 5인것을 확인할 수 있다.


### int[] (primitive type)

```java
private static void ListToArrayInt() {
    List<Integer> list = Arrays.asList(1, 2, 3);

    //  int[] arr = list2.toArray(new Integer[0]);   // primitive 불가능
    int[] arr = new int[list.size()];
    for (int i = 0; i < list.size(); ++i) {
        arr[i] = list.get(i);
    }
}
```

- primitive 타입으로는 별 방법이 없다. 직접 하나씩 넣어주어야한다.


---

## Reference
- https://stackoverflow.com/questions/9572795/convert-list-to-array-in-java
- http://asuraiv.blogspot.com/2015/06/java-list-toarray.html