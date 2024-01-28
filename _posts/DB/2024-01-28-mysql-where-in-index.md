---
title: WHERE IN 사용시 INDEX 를 타지않을 수 있다.
date: 2024-01-28-13:00
categories:
- DB

tags:
- DB
- DataBase
- MYSQL
- TroubleShooting

---

## SQL을 작성할때 WHERE IN 사용시 인덱스를 타지 않는 경우를 알아봅니다.
> WHERE IN 에 들어가는 조건이 많으면 인덱스를 타지 않습니다.

---

## Goal
- SQL을 작성할때 인덱스가 걸려있는 컬럼에 WHERE IN 사용시 인덱스를 타지 않는 경우를 알아봅니다.
- 해결하는 방법을 알아봅니다.

---

## 조건에 들어가는 파라메터들의 메모리 사이즈가 일정 사이즈를 넘으면 안된다.
> 일반적으로 1000개 이하로 끊는것이 좋습니다.

- [공식문서](https://dev.mysql.com/doc/refman/5.7/en/range-optimization.html) 에 따르면 파라메터의 개수가 `range_optimizer_max_mem_size` 옵션을 넘어서면 인덱스를 타지 않고 테이블 full scan 을 한다고 명시되어있습니다.
- 즉 조건에 들어가는것이 String 이냐, Long 이냐 Int 이냐 등등 타입형태에 따라서 같은 개수이지만 인덱스를 탈수도 있고, 아닐수도 있습니다.
- 일반적으로는 **1000개** 이하정도라면 인덱스를 무난히 타게됩니다.

> To control the memory available to the range optimizer, use the [range_optimizer_max_mem_size](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_range_optimizer_max_mem_size) system variable:
> A value of 0 means “no limit.”
> With a value greater than 0, the optimizer tracks the memory consumed when considering the range access method. If the specified limit is about to be exceeded, the range access method is abandoned and other methods, including a full table scan, are considered instead. This could be less optimal. If this happens, the following warning occurs (where N is the current [range_optimizer_max_mem_size](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_range_optimizer_max_mem_size) value):

```
Warning    3170    Memory capacity of N bytes for
                   'range_optimizer_max_mem_size' exceeded. Range
                   optimization was not done for this query.
```


### 주로 ORM 을 사용하는 부분에서 발생하기 쉽습니다.

```kotlin
// case1
fun findAllByUuids(uuids: List<Long>)

// case2
fun findAllBy(predicate: Predicate)
```

- 위와같은 코드를 사용하는경우 문제가 발생하기 쉽습니다.
- case1 같은 경우에는 호출하는쪽에서 몇개를 넣을지 판단하는 로직이 없다면 문제가 발생할 수 있는 여지가 있고.
- case2 같은 경우에는, 어떤 조건이 들어올지 모르기 떄문에 문제가 발생할 수 있는 여지가 있습니다. (애초에 이런 패턴은 지양해야한다고 개인적으로 생각하고 있습니다.)

---

## 해결방법
> chunk 처리, 옵션 설정

- 크게 두가지 방법으로 해결 할 수 있습니다.
  - Where In 호출하는쪽에서 Chunk 처리하기 (1000개 이하)
  - `range_optimizer_max_mem_size` 사이즈 늘리기


- 저의 경우에는 Repository 를 호출하는 Dao 레벨에서, chunk 로 잘라내게 코드를 작성하는것을 선호합니다.
- 이유는 애초에 많은 데이터를 가져오는 상황에서, `range_optimizer_max_mem_size` 옵션을 통해 한번에 대량 처리하는것이 한개의 네트워크 송수신 과정에서의 리스크가 더 높다고 판단하여, chunk 처리하여 하는것이 더 데이터를 안전하게 가져올 수 있다고 판단했습니다.
- 또한 `range_optimizer_max_mem_size` 을 기억해놓고 추후 어느정도 까지 처리할수 있음을 어림짐작하며 처리하거나, 계속 메모리 사이즈를 늘려가야하는 상황이 존재하는것을 선호하지 않는 부분도 있습니다.

## Conclusion
- 인덱스가 걸려있는 WHERE IN 절에 들어가는 조건들의 총 메모리 사이즈가 `range_optimizer_max_mem_size` 을 넘어가면 인덱스를 타지 못한다.
- `range_optimizer_max_mem_size` 를 늘리거나, chunk 호출 처리(일반적으로 1000개)로 해결할 수 있다.

---

## Reference
- https://dev.mysql.com/doc/refman/5.7/en/range-optimization.html
