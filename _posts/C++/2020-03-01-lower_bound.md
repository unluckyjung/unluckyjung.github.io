---
title: lower_bound , upper_bound 사용법
categories:
- C++

tags:
- C++
- STL
- lower_bound
- upper_bound
---

## lower_bound, upper_bound
> 오름 or 내림차순 정렬이 되어 있어야 한다

```c++
lower_bound(vec.begin(), vec.end(), num);
```

> 
이렇게 생각하면 쉽다.  `num`을 찾을때,  
`lower_bound` = `num` 보다 작지 않은 첫번째 원소.  
`upper_bound` = `num` 보다 큰 첫번째 원소.  

**둘다 찾지 못한다면, end() 리턴**

---

### 역방향으로도 가능하다.
**내림차순 정렬이 되어 있어야 한다.**

```c++
lower_bound(vec2.begin(), vec2.end(), num, greater<int>());
```

>
`lower_bound` = `num` 보다 크지 않은 첫번째 원소.  
`upper_bound` = `num` 보다 작은 첫번째 원소.  

**둘다 찾지 못한다면, end() 리턴**

---


### 예시

```c++
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main()
{
	vector<int> vec{ 1,3,5 };
	cout << *lower_bound(vec.begin(), vec.end(), 3) << "\n";
	cout << *upper_bound(vec.begin(), vec.end(), 3) << "\n";

	vector<int>::iterator it = lower_bound(vec.begin(), vec.end(), 6);
	if (it == vec.end()) {
		cout << "err\n";
	}

	cout << "\n";
	cout << "==reverse lower_bound==\n";
	vector<int> vec2{ 8,6,4,2 };
	cout << *lower_bound(vec2.begin(), vec2.end(), 3, greater<int>()) << "\n";
	return 0;
}
```

### 결과
```
3
5
err

==reverse lower_bound==
2
```