---
title: Permutation 사용시 주의할점.
date: 2020-04-18-23:00
categories:
- C++

tags:
- C++
- permutation

# photos: 
# - 주소

---

## permutation시 순서를 조심하자.
> prev와 next (reverse)는 다르다.

### prev vs next (reverse)
> 결과는 같지만, 과정이 다르다.

* `prev`의 경우, 내림차순으로 만들어간다.
* `next (reverse)`의 경우, 역순으로 보았을때 오름차순으로 만들어 간다.

* 결론적으로 만들어지는 순열의 조합의 종류와, 결과는 같다
* **하지만 만들어지는 과정이 다르다.**

---

### 코드

```c++
#include <iostream>
#include <algorithm>
#include <vector>

using namespace std;

int main()
{
	vector<int> v{ 3,2,1 };

	cout << "prev_permutation\n";

	do {
		for (auto it : v) {
			cout << it << " ";
		}
		cout << "\n";
	} while (prev_permutation(v.begin(), v.end()));

	v = { 3,2,1 };

	cout << "-----------------\n";

	cout << "next_permutation (reverse)\n";

	do {
		for (auto it : v) {
			cout << it << " ";
		}
		cout << "\n";

	} while (next_permutation(v.rbegin(), v.rend()));

	return 0;
}
```

### 결과

```
prev_permutation
3 2 1
3 1 2
2 3 1
2 1 3
1 3 2
1 2 3
-----------------
next_permutation (reverse)
3 2 1
2 3 1
3 1 2
1 3 2
2 1 3
1 2 3
```

### 해석

* **`prev`의 경우**
    * `3 2 1` 다음 작은 수열은 `3 1 2`

* **`next (reverse)`의 경우**
    * `3 2 1` 을 거꾸로 보면 `1 2 3` 이다.
    * `1 2 3` 의 다음 큰 수열은 `1 3 2`
    * 뒤집으면 `2 3 1` 이된다.



---

### 여담
> 지금껏 동작방식을 잘못 안 상태로 있었다.

* 모든 조합을 만들어보는 목적은 달성하고 있어서, 지금까지 문제를 발견하지 못했었다.
* 둘이 같은 결과가 나올꺼라고 생각하면서 써오고 있었다.