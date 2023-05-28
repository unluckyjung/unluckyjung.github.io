---
title: STL to Collection (Sort)
date: 2022-01-05-18:15
categories:
- Java
- Kotlin

tags:
- Java
- Kotlin
- Sort
- Collection

---

## STL to Collection (5)
> cpp에서 사용하는 정렬 관련기능을 java, kotlin 로 치환해 봅니다.

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

### Kotlin

```kotlin
val arrayNumber = arrayOf(1,5,3)    // 기존 객체 변화
arrayNumber.sort() // {1, 3, 5}     
```

```kotlin
val numbers = listOf(3, 4, 1, 2)    
numbers.sorted().forEach {  // 정렬된 새로운 객체 반환, 기존 numbers 객체는 비정렬 상태 유지
    println(it)
}
```


### 객체 정렬 Basic

```kotlin
data class SortMan(
    val age: Int,
    val grade: Int,
    val birthDayTime: ZonedDateTime = ZonedDateTime.now(),
)

println("age 오름차순 정렬")
val sortedMans1 = mans.sortedBy { it.age }
sortedMans1.forEach {
    println(it.toString())
}

println("age 오름차순 grade 내림차순")
val sortedMans2 = mans.sortedWith(
    compareBy({ it.age }, { -it.grade }),
)
```

- `sortedBy` 를 이용해 정렬조건을 넣어줄 수 있다.
- `sortedWith` 를 이용해 comparator 를 넣어줄 수도 있는데, 이때 `comparator` 를 `compareBy` 로 전달해 줄 수 있다.


---

## 역순 정렬

### cpp

```cpp
vector<int> nums = {1, 2, 3, 4};
sort(nums.begin(), nums.end(), greater<>()); // {4,3,2,1}

nums = {1, 2, 3, 4};
sort(nums.rbegin(), nums.rend()); // {4,3,2,1}
```

### java

```java
List<Integer> nums = Arrays.asList(1, 2, 3, 4);
Collections.sort(nums, Collections.reverseOrder());

nums = Arrays.asList(1, 2, 3, 4);
Collections.sort(nums, Comparator.reverseOrder());  // {4, 3, 2, 1}

nums = Arrays.asList(1, 2, 3, 4);
nums.sort(Collections.reverseOrder());  // {4, 3, 2, 1}
```

### kotlin

```kotlin
val numbers = listOf(3, 4, 1, 2)
numbers.sortedDescending().forEach {
    print(it)
}
```

```kotlin
println("age 내림차순 정렬1")
val sortedMans2 = mans.sortedBy { -it.age }
sortedMans2.forEach {
    println(it.toString())
}

println("age 내림차순 정렬2")
val sortedMans3 = mans.sortedByDescending { it.age }
sortedMans3.forEach {
    println(it.toString())
}
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

### kotlin
- kotlin 의 경우에는 기본 sort 역시 **stable** sort 입니다. [docs](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/sort.html)

> The sort is stable. It means that equal elements preserve their order relative to each other after sorting.


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

```kotlin
val numbers = listOf("12", "034")
numbers.sorted().forEach {
    println(it) // {"034", "12"}
}
```

- cpp와 마찬가지로 숫자형태를 가진 문자열인 경우, 앞에 `"0"` 이 있는 경우를 주의해야 합니다.

---

## 객체, 구조체정렬

### java
- 위에서 셜명한 방법인 `Comparator` 와 `static method`, `Comparable` 를 사용하는 방법이 있습니다. [STL to Collection (PriorityQueue) 에 적은 내용으로 대체합니다.](https://unluckyjung.github.io/java/kotlin/2021/12/21/STL-To-Collection-Queue-Deque-PriorityQueue/)
- 정렬 조건이 여러개인 경우는 [이 글](https://unluckyjung.github.io/java/2021/02/23/Java-Custom-Sort/)로 대체합니다.


### kotlin
> 여러 조건을 넣어 정렬

```kotlin
private fun multiCompare(
    mans: List<SortMan>,
) {
    println("age 오름차순 grade 내림차순")
    val sortedMans2 = mans.sortedWith(
        compareBy({ it.age }, { -it.grade }),
    )

    sortedMans2.forEach {
        println(it.toString())
    }

    println("age 오름차순 birthDayTime 내림차순")
    val sortedMans4 = mans.sortedWith(
        compareBy<SortMan> { it.age }.thenByDescending { it.birthDayTime },
    )

    sortedMans4.forEach {
        println(it.toString())
    }

    println("age 오름차순 birthDayTime 내림차순")
    val sortedMans5 = mans.sortedWith(
        Comparator<SortMan> { m1, m2 ->
            if (m1.age == m2.age) {
                -m1.birthDayTime.compareTo(m2.birthDayTime)
            } else {
                m1.age.compareTo(m2.age)
            }
        }
    )

    sortedMans5.forEach {
        println(it.toString())
    }
}
```

- `thenByDescending` 을 이용해서 음수 연산이 불가능한 경우(ZoneDateTime) 도 내림차순으로 처리해줄 수 있습니다.

---

## Reference
- [Comparable, Comparator](https://docs.oracle.com/javase/8/docs/api/)
- [Baeldung](https://www.baeldung.com/java-8-sort-lambda)
- [Kotlin Code](https://github.com/unluckyjung/kotlin-learning/blob/f89fc0bc396cc6c76c187f7b37785dcfb5e88ed5/kotlin-learning/src/main/kotlin/basic/syntax/ps/KtSort.kt)
- [Kotlin docs](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/sort.html)