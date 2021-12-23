---
title: STL to Collection (Queue, Deque, PriorityQueue)
date: 2021-12-21-20:30
categories:
- Java

tags:
- Java
- Collection

---

## STL to Collection (2)
> cpp에서 사용하는 Queue, Deque, PriorityQueue 을 java로 치환해봅니다.

---

# 큐 메소드별 동착 차이
> 실패시의 반환값이 다릅니다.

| 동작| 예외 발생 | 값 반환 |
|:---:|:---:|:---:|
|값 삽입 | add(e) | offer(e) / true or false|
|값 제거 | remove() | poll() / element or null |
|값 확인 | element() | peek() / element or null |

> 참고: LinkedList 구현체에서는 null 삽입이 가능합니다.

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

# Queue
> cpp


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

> java


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

> cpp


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

> java


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

> cpp

```cpp
priority_queue<int, vector<int>> pq;
pq.push(10);
cout << pq.top() << "\n"; // (10)
cout << pq.size() << "\n";  // (1)

cout << pq.empty() <<  "\n";  // (false)
pq.pop();
cout << pq.empty()  << "\n";  // (true)
```


> java

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
### 기본 정렬 구조

- Cpp의 경우에는 기본이 **MaxHeap**, 자바의 경우에는 **MinHeap** 으로 작동한다.

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

### 역순 정렬

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

---

## 만든 구조체 or 객체 정렬 기준을 세워주기

> cpp

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

> java

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

> 람다 버전

```java
public static void main(String[] args) {
    PriorityQueue<Man> pq = new PriorityQueue<>((o1, o2) -> -Integer.compare(o1.getAge(), o2.getAge()));  // Max Heap (음수이므로)
    pq.offer(new Man(20, "aaa"));
    pq.offer(new Man(30, "bbb"));

    System.out.println(pq.peek().toString());   // Man{age=30, name='bbb'}
}
```

---
