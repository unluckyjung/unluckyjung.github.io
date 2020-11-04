---
title: 프로그래머스 타겟 넘버 [C++]
date: 2020-11-03-17:00
categories:
- PS

tags:
- Programmers
- PS
- Problem Solve
- DFS

---

## Problem : [타겟 넘버](https://programmers.co.kr/learn/courses/30/lessons/43165)
> 유형 : DFS

---
 
## 문제 해석
* 벡터 안의 수를 `전부` 이용하여 빼기, 더하기 연산을 통해 타겟넘버를 만드는 경우의 수를 구하라.

## 해결 전략
* 주어지는 숫자의 개수가 최대 20개이다
  * 모든 경우를 시도 해보면 된다.
  * `2^20-1` 의 복잡도를 가진다.

## 구현
* `DFS` 를 이용해 전부 탐색해본다.
* 전부 계산 했을 때 (인덱스가 끝에 도달했을때) 결과를 보고, 타겟 넘버인지를 확인한다.
* 계산 할것이 남아 있는 경우, 빼거나, 더하거나 두가지 경우로 DFS를 이어 진행한다.


---

## 코드

```c++
#include <string>
#include <vector>

using namespace std;
int answer, numbersLen;
vector<int> numbers;

void dfs(int index, int sum, int target){
    if(index == numbersLen){
        if(sum == target) answer++;
        return;
    }
    dfs(index + 1, sum + numbers[index], target);
    dfs(index + 1, sum - numbers[index], target);
}


int solution(vector<int> nums, int target) {
    numbers = nums;
    numbersLen = numbers.size();
    
    dfs(0, 0, target);
    
    return answer;
}
```

---

## 피드백
* 과거 알고리즘을 공부하던 초창기에 두번이나 풀어봤던 문제이다.
* 그때는 바로 솔루션을 내지 못했던거 같은데, 지금 보니 굉장히 쉽다. 
* 지금 어려운 문제들도 나중의 나에게는 쉬울 것이다. **화이팅**
