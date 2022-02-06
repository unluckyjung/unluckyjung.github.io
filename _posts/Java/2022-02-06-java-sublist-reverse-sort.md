---
title: Java List 부분 정렬 및 뒤집기
date: 2022-02-06-18:06
categories:
- Java

tags:
- Java

---

## List의 부분 정렬 및 뒤집기
> List의 일부분을 정렬해보거나, 뒤집어봅니다.

---

## Goal
- List의 일부분을 정렬하거나 뒤집어봅니다.

---

## 부분 정렬
> `subList(startIndex, endIndex)` 을 이용합니다. (end is exclusive)

```java
public static void main(String[] args) {
    List<Integer> list = Arrays.asList(1, 5, 3, 4, 2);
    Collections.sort(list.subList(0, 3));
    for (int num : list) {
        System.out.print(num + " ");    //  1 3 5 4 2
    }
}
```

- `list.subList(0, 3)` 을 이용해 얻어낸 부분 리스트만 정렬해봅니다.
- `0`번 인덱스에서 `2(3-1)`번 인덱스 구간인 `{1, 5, 3}` 부분만 정렬되는것을 확인 할 수 있습니다.



> The returned list is backed by this list, so non-structural changes in the returned list are reflected in this list, and vice-versa. The returned list supports all of the optional list operations supported by this list.


- **subList**의 경우 [공식문서](https://docs.oracle.com/javase/8/docs/api/java/util/List.html#subList-int-int-)에 따르면, 반환되는것이 실제 List의 형태입니다.
- 즉 `list.subList` 를 했다고 해서 새로운 객체가 반환되는것이 아니기 때문에, 부분 정렬이 정상적으로 수행됩니다.

```java
public static void main(String[] args) {
    List<Integer> originalList = Arrays.asList(1, 5, 3, 4, 2);
    List<Integer> subList = originalList.subList(0, 3);
    subList.set(0, 111);

    System.out.println(originalList.get(0));    // 111
}
```

- **subList**의 `0`번 인덱스를 `111`로 변환하지만, **originalList**의 `0`번 인덱스값도 **같이 변경**되는것을 확인할 수 있습니다.

## 부분 뒤집기

```java
public static void main(String[] args) {
    List<Integer> originalList = Arrays.asList(1, 5, 3, 4, 2);
    Collections.reverse(originalList.subList(0, 3));

    for (int num : originalList) {
        System.out.print(num + " ");    // 3 5 1 4 2
    }
}
```

- 부분 정렬 부분과 동일한 방식으로 구현해주면 됩니다.

---

## Reference
- [java 8 docs](https://docs.oracle.com/javase/8/docs/api/java/util/List.html#subList-int-int-)