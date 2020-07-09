---
title: pair <data type, data type> 의 크기는 몇일까?
categories: 
- Cpp

tags:
- C++
- VS
---

## pair<long long, int> 의 크기는 8 + 4 = 12인가요?
> 답부터 말하자면 아니다. 16이다.

* 단순히 생각하면 `long long (8byte)` + `int (4byte)` = 16 일꺼 같지만  
* pair 자료형은 구조체와 마찬가지로 패딩비트를 이용해, 큰 자료형에 크기를 맞춘다.  

> 패딩 비트가 뭔가요? 
[이거입니다](https://supercoding.tistory.com/37)

#### 예시를 보자.

```c++
#include <bits/stdc++.h>
using namespace std;

int main()
{
   pair<int, long long> pil;
   cout << sizeof(pil) <<"\n";

   pair<int, int> pi;
   cout << sizeof(pi) <<"\n";

   pair<long long,  long long> pll;
   cout << sizeof(pll) <<"\n";

   return 0;
}
```

```
16
8
16
```

---

#### 어? 그러면 이거 완전히 구조체와 같잖아?
* **네, 구조체로 만들어졌으니까요**

> pair의 정의를 찾아보니..

```c++
struct pair { // store a pair of values
    using first_type  = _Ty1;
    using second_type = _Ty2;
```

---

> 의문이 생기면, 찾아보자. 이것은 나중에 나에게 도움이 될 경험치가 될것이다.