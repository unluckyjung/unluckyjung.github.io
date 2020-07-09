---
title: C++ Map을 value 기준으로 정렬하기
date: 2020-05-07-11:50
categories:
- Cpp

tags:
- Cpp
- map
- sort

# photos: 
# - 주소

---

## map을 key값이 아닌 value 기준으로 정렬해봅니다.
> map의 **요소**들을 `value`값을 기준으로 정렬해봅니다.

---

## Map의 기본 정렬상태
> map은 `key`값을 기준으로 내림차순 정렬되어 있습니다.

* 오름차순으로 정렬하기를 원한다면 
* key값이 int 인경우 맵을 선언시 `gretaer<int>` 비교 함수를 넣어주면 됩니다.

## value값으로 정렬하기
> 정확히는 `map`을 정렬하는 것이 아니고, map의 **요소**들을 `value`값을 기준으로 정렬합니다.

* 두단계로 진행이 되어야합니다.
* [1] `map`을 `vector`로 이동
* [2] `vector`를 **second** 기준으로 정렬

---

### map을 vector로 이동
> map의 key value가 `<int, int>` 일때

```c++
vector<pair<int,int>> vec( m.begin(), m.end() );
```

<br>

### value 기준 비교 함수 작성
> map의 key value가 `<int, int>` 일때

```c++
bool cmp(const pair<int,int>& a, const pair<int,int>& b) {
	if (a.second == b.second) return a.first < b.first;
	return a.second < b.second;
}
```

<br>

### 정렬
> map의 key value가 `<int, int>` 일때

```c++
sort(vec.begin(), vec.end(), cmp);
```


---

## 예제 코드

```c++
#include<iostream>
#include<algorithm>
#include<vector>
#include<map>

#define pp pair<int,int>

using namespace std;

map<int, int> m;

bool cmp(const pp& a, const pp& b) {
	if (a.second == b.second) return a.first < b.first;
	return a.second < b.second;
}

void Map_Setting() {
	for (int i = 0; i < 1; ++i) m[1]++;
	for (int i = 0; i < 5; ++i) m[2]++;
	for (int i = 0; i < 3; ++i) m[3]++;
}

int main() {

	Map_Setting();
	for (auto num : m) {
		cout << "key: " << num.first << " | value: " << num.second << "\n";
	}

	cout << "\n=======sort========\n\n";

	vector<pp> vec( m.begin(), m.end() );
	sort(vec.begin(), vec.end(), cmp);

	for (auto num : vec) {
		cout << "key: "<< num.first << " | value: " << num.second << "\n";
	}

	return 0;
}

```

---

## 결과

```
key: 1 | value: 1
key: 2 | value: 5
key: 3 | value: 3

=======sort========

key: 1 | value: 1
key: 3 | value: 3
key: 2 | value: 5
```

---

### Reference

* http://www.cplusplus.com/reference/