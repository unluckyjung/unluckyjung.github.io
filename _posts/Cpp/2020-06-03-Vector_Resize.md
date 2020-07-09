---
title: C++ vector resize 시 주의할 점
date: 2020-06-16-18:57
categories:
- Cpp

tags:
- Cpp
- vector
- resize

# photos: 
# - 주소

---

## Vector의 `resize`는 벡터의 크기를 재할당 합니다.
> `resize`시 주의할점을 알아 봅니다.

---

## resize를 하면서, 원소 채워넣기
> :heavy_check_mark: `vector.resize(size, value)`

* 하지만 이때, 기존보다 더 작게 할당하게 될경우 어떻게 될까 궁금해졌다
* 만약 `1` 라는 값을 `5` 개 가진 벡터를
* `3` 의 크기로 `resizing` 하며 `2` 로 채운다면?

---

## 코드

```c++
#include<iostream>
#include<vector>

using namespace std;

void Print_Vec(const vector<int> vec) {
	for (auto const &it : vec) {
		cout << it << " ";
	}
	cout << "\n";
}

int main() {
	vector<int> vec(5, 1);
	Print_Vec(vec);

	vec.resize(10, 2);
	Print_Vec(vec);

	vec.resize(3, 3);
	Print_Vec(vec);

	return 0;
}
```

---

## 결과

```
1 1 1 1 1
1 1 1 1 1 2 2 2 2 2
1 1 1
```

* `resize` 하며 더 커지는 경우에만, 값이 들어가는 것을 볼 수 있다.


---


### Reference

* http://www.cplusplus.com/reference/