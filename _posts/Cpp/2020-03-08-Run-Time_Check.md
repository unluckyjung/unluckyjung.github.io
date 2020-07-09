---
title: Run-Time Check Failure `#2 - Stack around the variable` 해결
categories: 
- Cpp

tags:
- Cpp
- VS
---

## Run-Time Check Failure `#2 - Stack around the variable
> PS를 하다가, 처음보는 에러를 발견했다.  
어떤 에러인지 찾아보았다.

> 사용자가 설정한 크기보다 더 큰 데이터를 넣으려고 할때 발생한다.  
자료형이거나, 배열 크기라던가..

#### 나의 경우 다음과 같은 경우에서 에러가 발생했었다.

```c++
int map[12][12];
int copy_map[10][10];

memcpy(copy_map, map, sizeof(map));

```

* `copy_map` 보다 더큰 데이터인 `map`을 넣으려고 했기 때문에 오류가 발생했었다.


---

> 항상 자료형, 크기에 조심하자