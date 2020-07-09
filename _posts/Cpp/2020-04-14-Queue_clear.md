---
title : C++ 큐 초기화 하기
date : 2020-04-14-17:50
categories : 
- Cpp

tags : 
- Cpp
- queue

---

## C++ 에서 Queue를 쉽게 비우는 방법
> 참고로 STL `queue`에서는 `vector`와 다르게 clear가 없습니다.

---

## 이미 알고 계신분들이 많을 수 있다.
> 하지만 저는 몰랐습니다.

#### Before : 큐를 비우던 방법
> 큐가 빌때까지 pop 을 했습니다.

```c++
while(!q.empty())   q.pop();
```

#### After : 큐를 재선언하여 비우기
> 정말 별거 없습니다.

```c++
q = queue<int>()
```

---

### 정말 저렇게 되냐구요?
> 네

#### 소스

```c++
#include <iostream>
#include <queue>

using namespace std;

int main()
{
    queue<int> q;
    q.push(1);
    cout << q.size() << "\n";

    q = queue<int>();
    cout << q.size() << "\n";
    
    return 0;
}
```

#### 결과

```
1
0
```

---

> 정말 별거 아니지만, 저런식으로 초기화 하는것을 처음 보아서 기록해 보았습니다.