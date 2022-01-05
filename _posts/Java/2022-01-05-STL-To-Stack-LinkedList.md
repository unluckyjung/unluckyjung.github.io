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

---


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

## (참고) 이터레이터 사용법

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

## Reference
- [Stack](https://docs.oracle.com/javase/8/docs/api/java/util/Stack.html)
- [LinkedList](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedList.html)