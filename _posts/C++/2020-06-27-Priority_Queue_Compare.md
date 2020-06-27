---
title: 구조체 priority queue 를 사용하기
date: 2020-06-27-23:00
categories:
- C++

tags:
- C++
- priority queue
- struct

---

## 구조체 타입을 priority queue에 넣어봅니다.
> `priority queue` 는 `heap` 자료구조로, 비교 연산이 항상 되어야 합니다.

---

## 기본 구조
> 우선순위 큐의 기본 형태는 다음과 같습니다.

* 기본적으로는 `MAX heap` 으로 작동합니다.

```c++
priority_queue<int, vector<int>, greater<int>> g_pq;  //작은것을 뽑고 싶을때
priority_queue<int, vector<int>, less<int>> l_pq; // 큰것을 뽑고 싶을때
```

## 구조체를 priority queue에 넣기
> 구조체 원소들간에 `대소 비교`가 되어야 합니다.

* `클래스`를 하나 만들고, 오버로딩을 해줍니다.

```c++
struct compare {
	bool operator()(const Struct& s1, const Struct& s2) {
		return s1.compare_value < s2.compare_value;
	}
};

priority_queue<Struct, vector<Struct>, compare> pq;
```

* 이 방법을 통해 필요에 맞춰서 오버로딩이 가능합니다.

---

## 예시
> 남자의 정보를 담는 구조체를 `heap` 에 넣어 보기

* 나이를 기준으로 우선순위를 정해줍니다.

### 코드
```c++
#include <queue>
#include <iostream>
using namespace std;

struct man {
	int age;
	int h, w;
};

struct compare {
	bool operator()(const man& m1, const man& m2) {
		return m1.age < m2.age;
	}
};

priority_queue<man, vector<man>, compare> mq;

int main()
{
	man tmp;
	tmp.age = 40;
	mq.push(tmp);

	tmp.age = 50;
	mq.push(tmp);

	tmp.age = 30;
	mq.push(tmp);

	tmp.age = 1;
	mq.push(tmp);

	while (!mq.empty()) {
		cout << mq.top().age << " ";
		mq.pop();
	}

	return 0;
}
```

### 결과

```
50 40 30 1
```

---