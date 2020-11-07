---
title: 프로그래머스 여행 경로 [C++]
date: 2020-11-06-16:00
categories:
- PS

tags:
- Programmers
- PS
- Problem Solve
- 완전탐색
- DFS
- 해시

---

## Problem : [단어 변환](https://programmers.co.kr/learn/courses/30/lessons/43164)
> 유형 : DFS, 해시

---
 
## 문제 해석
* 주어진 항공권을 전부 사용하여, ICN을 시작으로 하여 공항을 전부 순회하는 경로를 구하라.
  * 이때 알파벳 순서가 앞서는 경로를 우선으로 한다.

## 해결 전략
* 그래프로 생각하고 해결하였다.
* 공항과 공항의 관계를 단방향 그래프로 표현 한뒤, `DFS`를 통해 사용하지 않은 티켓들을 전부 사용해본다.
* 짧은 알파벳 순서부터 방문 해보아야 하므로, 한번 정렬을 해주었다.
* 경로의 경우 스택을 이용하여 보관하였다.

## 구현
* A에 연결된 다른 공항들의 집합이 필요했다.
  * Map과 Vector를 이용하였다.
  * `unordered_map<string, vector<string>> connectMap;`

* A에서 B로 가는 티켓이 존재하는지 확인을 할 필요가 있었다.
  * `[string][string]` 형태가 필요하여 `map` 자료구조를 사용하였다.
    * `unordered_map<string, unordered_map<string, int>> leftTicketCount;`
    * `["A"]["B"]` 로 가는 항공 티켓이 몇개나 있는지를 저장하였다.

* dfs 리턴값을 bool로 주어 최적의 경로를 찾은경우 모든 재귀를 종료하게 했다.
* 경로는 `answer` 에 스택의 개념을 활용하여 저장했다.


---

## 주의할점
* 같은 티켓이 여러장일 수 있다.

---

## 코드

```c++
#include <bits/stdc++.h>

using namespace std;
vector<string> answer;

unordered_map<string, vector<string>> connectMap;
unordered_map<string, unordered_map<string, int>> leftTicketCount;
int maxTicektCount;

bool dfs(string startAirport, int usedTicketCount) {
    if (maxTicektCount == usedTicketCount) {
        return true;
    }

    for (string Airport : connectMap[startAirport]) {

        if (leftTicketCount[startAirport][Airport] == 0) continue;
        leftTicketCount[startAirport][Airport]--;

        answer.push_back(Airport);
        if (dfs(Airport, usedTicketCount + 1)) return true;
        answer.pop_back();
        leftTicketCount[startAirport][Airport]++;
    }
    return false;
}

vector<string> solution(vector<vector<string>> tickets) {
    maxTicektCount = tickets.size();
    for (auto ticket : tickets) {
        connectMap[ticket[0]].push_back(ticket[1]);
        leftTicketCount[ticket[0]][ticket[1]]++;
    }

    for (auto ticket : tickets) {
        sort(connectMap[ticket[0]].begin(), connectMap[ticket[0]].end());
    }

    answer.push_back("ICN");
    dfs("ICN", 0);
    return answer;
}
```

---


## 피드백
* 같은 티켓이 여러장인 경우를 생각하지 못하고 bool로 관리하는 실수를 했다.
* 수도 코드를 좀 더 깔끔하게 쓸 필요가 있다.
