---
title: C++ multiset 에서 한개만 지우기
date: 2020-06-03-18:00
categories:
- C++

tags:
- C++
- multiset
- set

# photos: 
# - 주소

---

## C++의 `multiset` 에서 한개의 원소만 지워봅니다.
> `multiset` 은 중복된 원소를 담을 수 있는 이진트리 자료구조 입니다.

---

## 일반적으로 생각하는 방법 
> :x: `multiset.erase(val)`

* 이 방법은 `val`에 일치하는 `모든 노드`를 삭제 하게 됩니다.
* 이 경우 의도하지 않은 결과를 낳게 됩니다.


## 실제로 해야하는 방법 
> :heavy_check_mark: `iterator` : 반복자를 이용합니다.

* 원하는 원소가 등장하는 `iterator`를 찾습니다.
	* `md.find(val)`
	* `md.lower_bound(val)`
* 반복자를 이용해 삭제합니다.
* 물론 찾는것이 없는 경우에 대해서 예외처리가 필요합니다. 

---

## 코드

```c++
#include <iostream>
#include <set>
using namespace std;

void Erase_Val(multiset<int> ms) {    // { 1, 1, 2 }

    ms.erase(1);    // { 2 }
    cout << "Erase_Val result : " << ms.size() << "\n";
}

void Erase_Iter(multiset<int> ms) {    // { 1, 1, 2 }

    multiset<int>::iterator it = ms.find(1);
    ms.erase(it);   // { 1, 2 }
    cout << "Erase_Val result : " << ms.size() << "\n";
}

int main()
{
    multiset<int> ms{ 1,1,2 };
    Erase_Val(ms);
    Erase_Iter(ms);

    return 0;
}
```

---

## 결과

```
Erase_Val result : 1
Erase_Val result : 2
```

---

### Reference

* http://www.cplusplus.com/reference/