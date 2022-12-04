---
title: STL to Collection (Queue, Deque, PriorityQueue)
date: 2021-12-21-20:30
categories:
- Java
- Kotlin

tags:
- Java
- Kotlin
- Collection

---

## STL to Collection (2)
> cpp에서 사용하는 Queue, Deque, PriorityQueue 을 java, kotlin 으로 치환해봅니다.

---

## 메소드별 반환값에 차이가 있습니다.
> 실패시의 예외를 발생시키느냐, null 혹은 false를 반환하느냐의 차이가 있습니다.

| 동작| 예외 발생 | 값 반환 |
|:---:|:---:|:---:|
|값 삽입 | add(e) | offer(e) / true or false|
|값 제거 | remove() | poll() / element or null |
|값 확인 | element() | peek() / element or null |

> 참고: `LinkedList` 구현체에서는 null 삽입이 가능합니다.

```java
public class Main {
    public static void main(String[] args) {
        Queue<Integer> q = new LinkedList<>();
        q.offer(null);
        System.out.println(q.size());   // (1)
    }
}
```

- [공식문서 링크](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html)

---

# Queue

## cpp

```cpp
queue<int> q;
q.push(10); // {10}
q.push(20); // {10, 20}

cout << q.front() << "\n";  // (10)
cout << q.back() << "\n";   // (20)

q.pop(); // {20}

cout << q.front();
q.pop();

if (q.empty()) {
    cout << "empty" << "\n";    // "empty"
}
```

## java

```java
Queue<Integer> q = new LinkedList<>();
q.add(10);  //  실패시 예외발생
q.offer(20);    // 실패시 false 반환

System.out.println(q.element());    // (10), 없는 경우 예외 발생
System.out.println(q.peek());   // (10) , 없는 경우 null반환

System.out.println(q.poll());   // (10) pop + peek
System.out.println(q.remove()); // (20) 없는경우 예외 발생
System.out.println(q.poll());   // 출력: null,  {}

if(q.isEmpty()){
    System.out.println("empty");    // "empty"
}
```

---

# Deque

## cpp

```cpp
deque<int> dq = {10, 20, 30};
cout << dq.front() << "\n";  // 10;
cout << dq.back() << "\n";   // 30;

dq.pop_back(); // {10, 20};
dq.pop_front(); // {20};
dq.push_front(100); // {100, 20}
dq.push_back(200);  // {100, 20, 200};

cout << dq[1] << "\n";  // 20;
while(!dq.empty()){
    dq.pop_back();
}

if (dq.empty()) {
    cout << "empty" << "\n";    // "empty"
}
```

## java

```java
Deque<Integer> dq = new LinkedList<>();
dq.add(20); // {20}
dq.addLast(30); // {20,30}
dq.addFirst(10);    //  {10, 20, 30}
//dq.offer(N);

System.out.println(dq.peek());  // (10)
System.out.println(dq.peekFirst()); // (10)
System.out.println(dq.peekLast());  // (30)
//dq.element();


System.out.println(dq.pollFirst()); // (10)
System.out.println(dq.pollLast()); // (30)

System.out.println(dq.pop());   // (20) queue기반이라 frontPOP
// dq.poll();

if(dq.isEmpty()){
    System.out.println("empty");    // "empty"
}
```

---

# PriorityQueue

## 기본 문법

### cpp

```cpp
priority_queue<int, vector<int>> pq;
pq.push(10);
cout << pq.top() << "\n"; // (10)
cout << pq.size() << "\n";  // (1)

cout << pq.empty() <<  "\n";  // (false)
pq.pop();
cout << pq.empty()  << "\n";  // (true)
```


### java

```java
PriorityQueue<Integer> pq = new PriorityQueue<>();
pq.offer(10);
//pq.add(10);

System.out.println(pq.peek());  // top (10)
//System.out.println(pq.element());

System.out.println(pq.poll());  // top + pop (10)
//System.out.println(pq.remove());

pq.offer(20);
System.out.println(pq.size());  // (1)
System.out.println(pq.isEmpty());   // (false)
System.out.println(pq.poll());  // top + pop (20)
System.out.println(pq.isEmpty());   // (true)
```

## 기본 정렬 구조
> Cpp의 경우에는 기본이 **MaxHeap**, java, kotlin 의 경우에는 **MinHeap** 으로 작동한다.

```cpp
priority_queue<int, vector<int>> pq;    // Max Heap
pq.push(10);
pq.push(7);
cout << pq.top() << "\n"; // (10)
```

```java
PriorityQueue<Integer> pq = new PriorityQueue<>();  // Min Heap
pq.offer(10);
pq.offer(7);
System.out.println(pq.peek()); // (7)
```

```kotlin
val pq = PriorityQueue<Int>()
pq.add(10)
pq.add(7)
println(pq.element())   // (7)
```

## 역순 정렬
> cpp Max to Min

```cpp
priority_queue<int, vector<int>, greater<int>> pq;  // Min Heap으로 변경
pq.push(10);
pq.push(7);
cout << pq.top() << "\n"; // (7)
```

> java Min to Max

```java
PriorityQueue<Integer> pq = new PriorityQueue<>(Comparator.reverseOrder());  // Max Heap으로 변경
pq.offer(10);
pq.offer(7);
System.out.println(pq.peek()); // (10)
```

```kotlin
val pq = PriorityQueue<Int>(Comparator.reverseOrder())
pq.add(10)
pq.add(7)
println(pq.element())   // (10)
```

---

## 만든 구조체 or 객체 정렬 기준을 세워주기

### cpp

```cpp
#include <bits/stdc++.h>

using namespace std;

struct Man {
    int age;
    string name;
};

struct cmp {
    bool operator()(const Man &m1, const Man &m2) {
      return m1.age < m2.age;
    }
};

int main() {
  ios_base::sync_with_stdio(false);
  cin.tie(0);

  priority_queue<Man, vector<Man>, cmp> pq;  // Max Heap (나이가 제일큰게 top으로 간다)

  pq.push({29, "aaa"});
  pq.push({30, "bbb"});

  cout << pq.top().age << pq.top().name; // (30, "bbb")
}
```

### java(1)
> Comparable 인터페이스를 상속해 구현하는 방법

- 기본적인 정렬방식을 정의해준다.  

```java
class Man implements Comparable<Man> {
    @Override
    public int compareTo(final Man man) {
        // 기본적으로 MinHeap이기 때문에, MAXHeap을 위해 (-) 연산을 붙여주었다.
        return -Integer.compare(this.age, man.age);
    }
}
```


```java
import java.util.PriorityQueue;

public class Main {

    public static void main(String[] args) {
        PriorityQueue<Man> pq = new PriorityQueue<>();  // Min Heap
        pq.offer(new Man(20, "aaa"));
        pq.offer(new Man(30, "bbb"));

        System.out.println(pq.peek().toString());   // Man{age=30, name='bbb'}
    }
}
```

```java
class Man implements Comparable<Man> {

    private int age;
    private String name;

    public Man(final int age, final String name) {
        this.age = age;
        this.name = name;
    }

    @Override
    public int compareTo(final Man man) {
        // 기본적으로 MinHeap이기 때문에, MaxHeap을 위해 (-) 연산을 붙여주었다.
        return -Integer.compare(this.age, man.age);
    }
}
``` 

### java(2)
> Comporator를 이용하는방법  


- 내가 클래스를 직접 건들 수 없는 경우에 사용된다.
- 혹은 클래스에 정의된 기본 정렬방식과는 다른 정렬기준이 잠시 필요한경우 사용한다.

```java
public static void main(String[] args) throws IOException {
    PriorityQueue<Man> pq = new PriorityQueue<>(new Comparator<Man>() {
        @Override
        public int compare(final Man o1, final Man o2) {
            return -Integer.compare(o1.getAge(), o2.getAge());
        }
    });
    pq.offer(new Man(20, "aaa"));
    pq.offer(new Man(30, "bbb"));

    System.out.println(pq.peek().toString());   // Man{age=30, name='bbb'}
}
```

### 람다를 이용해 간결화

```java
public static void main(String[] args) {
    PriorityQueue<Man> pq = new PriorityQueue<>((o1, o2) -> -Integer.compare(o1.getAge(), o2.getAge()));  // Max Heap (음수이므로)
    pq.offer(new Man(20, "aaa"));
    pq.offer(new Man(30, "bbb"));

    System.out.println(pq.peek().toString());   // Man{age=30, name='bbb'}
}
```

### Kotlin

> 기본 정렬 조건 세워 주기 (`Comparable` 구현)

```kotlin
data class Man(
    val age: Int,
    val grade: Int,
    val name: String,
) : Comparable<Man> {

    // 기본 정렬 조건 정의: 나이가 많고, 등급은 낮을수록 우선순위
    override fun compareTo(other: Man): Int {
        return compareValuesBy(
            this, other,
            { -it.age }, { it.grade }
        )
    }

//    override fun compareTo(other: Man): Int {
//        return if (-age.compareTo(other.age) == 0) {
//            grade.compareTo(other.grade)
//        } else {
//            -age.compareTo(other.age)
//        }
//    }
}
```

```kotlin
private fun maxHeap() {
    val pq = PriorityQueue<Man>() // age Max Heap
    pq.add(Man(10, 1, "jys"))
    pq.add(Man(20, 0, "unluckyjung"))
    pq.add(Man(20, 2, "goodall"))
    pq.add(Man(15, -1, "yoonsung"))
    pq.add(Man(15, 10, "fortune"))

    while (!pq.isEmpty()) {
        println(pq.remove())
    }

/*  Man(age=20, grade=0, name=unluckyjung)
    Man(age=20, grade=2, name=goodall)
    Man(age=15, grade=-1, name=yoonsung)
    Man(age=15, grade=10, name=fortune)
    Man(age=10, grade=1, name=jys)
    */
}
```

- `compareValuesBy` 를 통해서, 간결하게 배교 우선순위를 정해 줄 수 있다.


> 정렬조건 만들어주기 `Comparator` 삽입

```kotlin
val pq = PriorityQueue(
    compareBy<Man> {
        it.age
    }
)

val pq = PriorityQueue(
    compareBy<Man> ({it.age}, {it.grade})
)

val pq = PriorityQueue(
    Comparator<Man> { m1, m2 ->
        m1.age.compareTo(m2.age)
    }
)

val pq = PriorityQueue { m1: Man, m2: Man ->
    m1.age.compareTo(m2.age)
}
```

1. Comparator 를 반환하는 `comapreBy<객체>{조건}` 으로 넣어주기
2. 직접 Comparator 를 정의하고 로직도 넣어주기
3. 2번을 람다식으로 풀어주기
