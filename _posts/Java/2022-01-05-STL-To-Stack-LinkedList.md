---
title: STL to Collection (Stack, LinkedList)
date: 2022-01-05-23:15
categories:
- Java

tags:
- Java
- Stack
- LinkedList
- Collection

---

## STL to Collection (6)
> cpp에서 사용하는 Stack, LinkedList 관련기능을 java로 치환해 봅니다.

---

# Stack

### cpp

```cpp
stack<int> s;
s.push(10); // {10}
s.push(20); // {20}
cout << s.top() << "\n";    // (20)
s.pop();    // {10}
cout << s.size() << "\n";   // (1)

s.pop();    // {}
if (s.empty()) {
    cout << "empty" << "\n";    // "empty"
}
```

### java


```java
Stack<Integer> s = new Stack<>();
s.push(10); // {10} 
s.push(20); // {10, 20}

System.out.println(s.peek());   //  (20)
s.pop();    // {10}
System.out.println(s.size());   //  (1)
s.pop();    //  {}

if (s.empty()) {
    System.out.println("empty");    // pass
}

if (s.isEmpty()) {
    System.out.println("empty");    // "empty"
}
```

### kotlin

```kotlin
val s = Stack<Int>()
s.push(10) // [10]
s.push(20) // [10, 20]

println(s.peek())   // (20)

println(s.firstElement()) // (10)
println(s.first())  // (10)

println(s.lastElement()) // (20)
println(s.last())   //(20)

println(s.pop())    // print 20, [10]
println(s.size) // 1

println(s.pop())    // print 10, []

if (s.isEmpty()) {
    println("empty") // "empty"
}
```

---

# LinkedList

<br>

### cpp

```cpp
#include <bits/stdc++.h>

using namespace std;

int main() {
  list<int> list1;

  list1.push_back(1); // {1}
  list1.push_back(3); // {1, 3}

  auto it = list1.begin();
  list1.insert(++it, 2);  // {1, 2, 3}
  it = list1.begin();
  list1.erase(it);  // {2, 3}
  list1.insert(list1.begin(), 10); // {10, 2, 3}

  cout << list1.front() << "\n"; // 10
  cout << list1.back() << "\n"; // 3

  return 0;
}
```

### java

```java
import java.util.LinkedList;

public class Main {

    public static void main(String[] args) throws IOException {


        LinkedList<Integer> list = new LinkedList();
        list.add(1);    // {1}
        list.add(3);    // {1, 3}

        list.add(1, 2);     // {1, 2, 3}

        list.remove(0); // {2, 3}   index remove 
        list.remove((Integer) 3);    // {2}     value(3) remove 값이 없다면 false 반환

        list.add(0, 10);    //  {10, 2};
        list.add(2, 3);    //  {10, 2, 3};
        list.add(2, 4);    //  {10, 2, 4, 3};

        System.out.println(list.getFirst());    // 10
        System.out.println(list.getLast()); // 3;

        System.out.println(list.get(2));    // 4

        // 원하는 값(2)의 인덱스를 찾은뒤 삭제하는경우
        int targetIndex = list.indexOf(2);   // 1;
        list.remove(targetIndex);   // {10, 4, 3};
    }
}
```

```kotlin
fun listBasic() {
    val list: LinkedList<Int> = LinkedList()
    list.add(1) // [1]
    list.add(3) // [1, 3]
    list.add(index = 1, element = 2) // [1, 2, 3]

    list.remove(element = 1) // [2, 3]
    list.remove(element = 3) // [2]
    println(list.remove(-1)) // false

    list.add(0, 10) //  [10, 2]
    list.add(2, 3) //  [10, 2, 3]
    list.add(2, 4) //  [10, 2, 4, 3]

    println(list.first) // 10
    println(list.last) // 3
    println(list[2]) // 4

    // 원하는 값(2)의 인덱스를 찾은뒤 삭제하는경우

    // [10, 2, 4, 3]
    val targetIndex = list.indexOf(2) // 1;
    list.removeAt(targetIndex)
    println(list)   // [10, 4, 3]
}
```

---

## 이터레이터 사용법

### cpp

```cpp
#include <bits/stdc++.h>

using namespace std;

int main() {

  list<int> list1 = {1, 2, 3};

  auto it = list1.begin();

  while (it != list1.end()) {
    cout << *it;  // 1 2 3
    it++;
  }

  return 0;
}
```


### java

```java
import java.util.Iterator;
import java.util.LinkedList;

public class Main {

    public static void main(String[] args) throws IOException {

        LinkedList<Integer> list = new LinkedList<Integer>() {
            {
                add(1);
                add(2);
                add(3);
            }
        };

        for (int num : list) {
            System.out.print(num);
            // 1 2 3
        }
        System.out.println();

        Iterator iter = list.iterator();

        while (iter.hasNext()) {
            System.out.print(iter.next());
            // 1 2 3
        }
        System.out.println();
    }
}
```
- **iterator** 조작이 cpp만큼 자유롭지는 않습니다.

### kotlin

```kotlin
val list = LinkedList<Int>().apply {
    addAll(mutableListOf(10, 20, 30))
}

val iter = list.iterator()
while (iter.hasNext()) {
    print("${iter.next()},")   // 10,20,30
}
println()
println()

val listIter = list.listIterator()

// [v,10,20,30]
println("prev index: ${listIter.previousIndex()}")   // -1
println("next index: ${listIter.nextIndex()}")   // 0
println("value: ${listIter.next()}")    // 10   [10,v,20,30]
println()

// [10,v,20,30]
println("prev index: ${listIter.previousIndex()}")   // 0
println("next index: ${listIter.nextIndex()}")   // 1
println("value: ${listIter.next()}")    // 20   [10,20,v,30]
println()

// [10,20,v,30]
println("prev index: ${listIter.previousIndex()}")   // 1
println("next index: ${listIter.nextIndex()}")   // 2
println("value: ${listIter.next()}")    // 30   [10,20,30,v]
println()

// [prev] [10,20,30,v]
println("prev index: ${listIter.previousIndex()}")   // 2
println("next index: ${listIter.nextIndex()}")   // 3
println("value: ${listIter.previous()}")    // 30   [10,20,v,30]
println()

// [prev] [10,20,v,30]
println("prev index: ${listIter.previousIndex()}")   // 1
println("next index: ${listIter.nextIndex()}")   // 2
println("value: ${listIter.previous()}")    // 20   [10,v,20,30]
println()

// [prev] [10,v,20,30]
println("prev index: ${listIter.previousIndex()}")   // 0
println("next index: ${listIter.nextIndex()}")   // 1
println("value: ${listIter.previous()}")    // 10   [v,10,20,30]
println()
```

- 기본적으로 `list.iterator()` 를 통해서 이터레이터를 얻을 수 있습니다.
- list 의 경우 `listIterator()` 를 이용하면, `next()`, `previous()` 를 이용해서 이러레이터를 앞뒤로 이동할 수 있습니다.

---

## Reference
- [Stack](https://docs.oracle.com/javase/8/docs/api/java/util/Stack.html)
- [LinkedList](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedList.html)