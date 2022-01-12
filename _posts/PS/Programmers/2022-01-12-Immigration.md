---
title: 프로그래머스 입국심사 [Java]
date: 2022-01-12-15:40
categories:
- PS

tags:
- Programmers
- PS
- Problem Solve
- 이분탐색
- Java

---

## Problem : [입국심사](https://programmers.co.kr/learn/courses/30/lessons/43238)
> 유형 : 이분탐색

---
 
## 문제해석
- 모든 사람이 심사를 받는데 걸리는 시간의 **최솟값**을 구하라

## 해결전략
- 사람의 수는 `1e8`, 심사하는데 걸리는 시간은 `1e9`분 까지 나올 수 있다.
- 걸리는 시간을 N으로 잡을때 `logN` 이하의 해결책을 구해야한다.
- **K 시간내에 해결할수 있는가?** 로 문제를 변형하고 **이분탐색**으로 확인한다.

## 구현
- 이분탐색의 **end** 지점을 제일 `느리게 일을 처리하는 심사관 * 기다리는 사람의 수` 로 잡는다.
- `주어진 시간 / 면접관의 처리속도` 를 통해서 해당 시간내에 모든 대기인원을 처리할수 있는지를 확인한다.

---

## 코드

```java
import java.util.Arrays;
import java.util.Collections;
import java.util.List;
import java.util.stream.Collectors;

class Solution {
    List<Integer> times;

    public long solution(int n, int[] timess) {
        times = Arrays.stream(timess).boxed().collect(Collectors.toList());
        Collections.sort(times);

        long answer = 0;
        long st = 1;
        long ed = (long) n * times.get(times.size() - 1);

        while (st <= ed) {
            long mid = st + (ed - st) / 2;
            if (isCan(n, mid)) {
                answer = mid;
                ed = mid - 1;
            } else {
                st = mid + 1;
            }
        }
        return answer;
    }

    private boolean isCan(final int targetCount, final long candidateTime) {
        long solHumanCount = 0;
        for (int speed : times) {
            solHumanCount += candidateTime / speed;
        }
        return solHumanCount >= targetCount;
    }
}
```
