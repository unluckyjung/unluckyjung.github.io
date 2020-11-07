---
title: C++ Map 비교하기
date: 2020-11-07-19:00
categories:
- Cpp

tags:
- Cpp

---

## C++ 의 Map을 비교해봅니다.
> `==` 로 판별이 가능합니다.

---

## 코드

```c++
#include <iostream>
#include <map>
using namespace std;

int main() {
	map<string, int> m1, m2, m3, m4, m5;
	m1["abc"] = 1;
	m2["abc"] = 1;
	m3["abc"] = 2;
	m4["ABC"] = 1;
	m5["ABC"] = 2;


	if (m1 == m2) cout << "m1, m2 Same\n";	// key 일치, value 일치
	if (m1 == m3) cout << "m1, m3 Same\n";	// key 일치, value 불일치
	if (m1 == m4) cout << "m2, m3 Same\n";	// key 불일치, value 불일치
	if (m1 == m5) cout << "m2, m5 Same\n";	// key 불일치, value 불일치

	m2["abcd"] = 1;
	if (m1 == m2) cout << "m1, m2 Same\n";	// key, value 가 추가되어 불일치

	
	// result : m1, m2 Same
	return 0;
}
```

* `key` 와 `value` 전부가 일치하는 경우에만 같다고 출력합니다.
* 물론 개수도 같아야 합니다.