---
title: STL to Collection (Sort)
date: 2022-01-05-18:15
categories:
- Java

tags:
- Java
- Sort
- Collection

---

## STL to Collection (5)
> cpp에서 사용하는 정렬 관련기능을 java로 치환해 봅니다.

## 기본 자료형 정렬

### cpp
```cpp
vector<int> nums = {4, 2, 3, 1};
sort(nums.begin(), nums.end());
```

### java(1) Collections.sort()
> `Collections.sort()`

```java
int[] unOrderedInts = {4, 3, 1, 2};
int[] ints = unOrderedInts.clone(); 

List<Integer> ints2 = Arrays.stream(ints).boxed().collect(Collectors.toList());
Collections.sort(ints2);    // 기본적으로 Integer 정렬
Collections.sort(ints2, Integer::compare);    // Comparator를 넣어줄수도 있습니다.
```

> java8 미만의 버전   


```java
Collections.sort(ints2, new Comparator<Integer>() {
    @Override
    public int compare(final Integer o1, final Integer o2) {
        return Integer.compare(o1, o2);
    }
});
```

### java(2) instance.sort()
> `instance.sort()`

```java
int[] unOrderedInts = {4, 3, 1, 2};

List<Integer> ints = new ArrayList<>();
for (int i : unOrderedInts) {
    ints.add(i);    // stream을 사용하지 않아도 됩니다.
}
ints.sort(Integer::compare);  // Collections.sort와는 다르게, 정렬 기준을 반드시 넣어주어야 합니다.
```

> java 8 미만의 버전  


```java
ints3.sort(new Comparator<Integer>() {
    @Override
    public int compare(final Integer o1, final Integer o2) {
        return Integer.compare(o1, o2);
    }
});
```

### java(3) static method 이용
> use static method

```java
public static void main(String[] args) {
    int[] unOrderedInts = {4, 3, 1, 2};
    List<Integer> ints3 = new ArrayList<>();
    for (int i : unOrderedInts) {
        ints3.add(i);
    }
    ints3.sort(Main::cmp);  // Main클래스 안의 static method인 cmp 호출
}

// static method 선언
private static int cmp(Integer o1, Integer o2) {
    return Integer.compare(o1, o2);
}
```

### java(4) Arrays.sort()
> `Arrays.sort()` 

```java
int[] unOrderedInts = {4, 3, 1, 2};
int[] ints = unOrderedInts.clone(); // 깊은 복사
Arrays.sort(ints);  // {1, 2, 3, 4}
```

- **사용하지 않는것을 권장합니다**
- **unstable**하며 최악의 경우 `O(N^2)` 의 시간복잡도를 가질 수 있습니다.
- **Double pivot quick sort**로 동작하기 때문에, 대신 **Tim Sort**로 동작하는 `Collections.sort()`를 사용하는것을 권장합니다.
- 이부분의 대한 내용은 따로 정리해서 소개할 예정입니다.

---

## 역순 정렬

### cpp

```cpp
vector<int> nums = {1, 2, 3, 4};
sort(nums.begin(), nums.end(), greater<>()); // {4,3,2,1}

nums = {1, 2, 3, 4};
sort(nums.rbegin(), nums.rend()); // {4,3,2,1}
```

```java
List<Integer> nums = Arrays.asList(1, 2, 3, 4);
Collections.sort(nums, Collections.reverseOrder());

nums = Arrays.asList(1, 2, 3, 4);
Collections.sort(nums, Comparator.reverseOrder());  // {4, 3, 2, 1}

nums = Arrays.asList(1, 2, 3, 4);
nums.sort(Collections.reverseOrder());  // {4, 3, 2, 1}
```

---

## Stable 정렬

### cpp

```java
vector<int> nums = {4, 2, 3, 1};
stable_sort(nums.begin(), nums.end());
```


### java
> 위에서 설명한 `Collections.sort()`를 사용하면 됩니다.

```java
int[] arr = {4, 2, 3, 1};
List<Integer> nums = new ArrayList<>();

for (int num : arr) {
    nums.add(num);
}

Collections.sort(nums);
```

---

## 문자열 정렬

> cpp 생략

```java
List<String> list = Arrays.asList("bb", "aaa");
Collections.sort(list);     // {"aaa", "bb"}
```

- 문자열 대소로 정렬합니다.

```java
List<String> list = Arrays.asList("12", "34");
Collections.sort(list);     // {"12", "34"}
```

```java
List<String> list = Arrays.asList("12", "034");
Collections.sort(list);     // {"034", "12"}
```
- cpp와 마찬가지로 숫자형태를 가진 문자열인 경우, 앞에 `"0"` 이 있는 경우를 주의해야 합니다.

---

## 객체, 구조체정렬

### java
- 위에서 셜명한 방법인 `Comparator` 와 `static method`, `Comparable` 를 사용하는 방법이 있습니다. [STL to Collection (PriorityQueue) 에 적은 내용으로 대체합니다.](https://unluckyjung.github.io/java/2021/12/21/STL-To-Collection-Queue-Deque-PriorityQueue/#%EB%A7%8C%EB%93%A0-%EA%B5%AC%EC%A1%B0%EC%B2%B4-or-%EA%B0%9D%EC%B2%B4-%EC%A0%95%EB%A0%AC-%EA%B8%B0%EC%A4%80%EC%9D%84-%EC%84%B8%EC%9B%8C%EC%A3%BC%EA%B8%B0)
- 정렬 조건이 여러개인 경우는 [이 글](https://unluckyjung.github.io/java/2021/02/23/Java-Custom-Sort/)로 대체합니다.

--

## Referenc
- [Comparable, Comparator](https://docs.oracle.com/javase/8/docs/api/)
- [Baeldung](https://www.baeldung.com/java-8-sort-lambda)