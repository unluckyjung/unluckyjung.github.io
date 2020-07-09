---
title: Vector vs Deque
categories: 
- Cpp

tags:
- Cpp
- STL
- vector
- deque
---

## Vector와 Deque는 비슷한것 같으면서도 매우 다르다.
> 가장 중요한 차이점은 메모리 에서의 연속성이다.

* `vector`는 메모리에 연속되어 있다
* `deque`는 메모리에 연속되어 있지 않다.

---

### Deque의 특징

> Deque

* **Deque**는 어느 방향으로도 `iterate`(참조) 가 가능하다.
    * `DQ[2]` 이런것이 가능하다.
* `vector`와 다르게, **양쪽** 끝 모두에서 원소 삽입, 삭제가 가능하다.
* 그리고, 중간에 `insert`, `erase`도 가능하다.

```c++
deque<int> DQ{1,2,3};
DQ.insert(DQ.begin() + 1 , 456);
DQ.erase(DQ.begin() + 2);
```

* **하지만** 연속되어 있는것처럼 보이지만 연속되어 있지 않다.
    * 메모리에 쪼개서 보관한다 `메모리 관리에 효율적`
        * 일관적인 성능을 보장한다.
    * 이 때문에 vector와는 다르게 `capacity` 와 `reserve` 가 없다.
    * 당연히 pointer 간 연산도 불가능하다.

---

### Vector의 특징

> Vector

* 메모리에 연속적으로 배치되어있다.
    * Cache Rate가 매우 높다.
* 중간에 원소를 삽입하거나 삭제하는 작업은 오래걸린다.
* 그리고 `capacity`는 기본적으로, `size`보다 크게 할당 되어진다.
    * 하지만 `capacity`가 부족한경우 **memory rellocate** 작업이 필요해진다.


---

### 시퀀스 컨테이너간 차이점

> list `노드` 기반

* 중간에 원소 삽입/삭제가 잦을때 매우 유리
* 개별 원소에 대한 접근이 `불가능`
* 메모리 오버헤드 높음

> vector `배열` 기반

* 개별 원소에 대한 접근이 잦을때 매우 유리
* 중간에 삽입/삭제시 매우 불리
* 큰 데이터를 다룰 경우에 불리

> deque `배열` 기반

* 개별 원소에 대한 접근이 잦을때 유리
* 중간에 삽입/삭제시 불리

---
> Reference

[모두의 코드](https://modoocode.com/176)  
[C++ 레퍼런스](http://www.cplusplus.com/reference/deque/deque/)