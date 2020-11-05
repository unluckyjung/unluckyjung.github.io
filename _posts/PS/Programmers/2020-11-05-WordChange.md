---
title: 프로그래머스 단어 변환 [C++]
date: 2020-11-05-16:00
categories:
- PS

tags:
- Programmers
- PS
- Problem Solve
- 완전탐색
- DFS

---

## Problem : [단어 변환](https://programmers.co.kr/learn/courses/30/lessons/43163)
> 유형 : DFS

---
 
 
## 문제 해석
* 시작 단어에서 벡터내 한 알파벳만 다른 단어로 교체하며 타겟 단어까지 가는 최소의 과정을 구하라.

## 해결 전략
* 모든 경우를 진행해본다.
* 단어의 최대길이는 10, 백터내 최대 단어의 개수는 50 이므로, 모든 경우를 해보아도 `(50*10)^2` 밖에 되지 않는다.

## 구현
* `DFS` 로 구현한다.
* 매번 벡터 전체를 순회하며, 이미 사용된 단어라면 넘어간다.
* 차이가 1이 나는 경우에만 `depth` 를 진행시키며 진행한다.
* 가능한 경우가 한번도 없다면, `INT_MAX` 이므로 이 경우에 대해서 예외처리해준다.




---

## 코드

```c++
#include <string>
#include <vector>
#include <climits>

using namespace std;
string targetWord;
vector<string> words;
int answer = INT_MAX;

bool visited[52];

bool isOnlyOneDiff(string w1, string w2) {
    int diffCount = 0;
    for (int i = 0; i < (int)w1.size(); ++i) {
        if (w1[i] != w2[i]) diffCount++;
    }

    if (diffCount == 1) return true;
    else return false;
}

void dfs(string curWord, int curCount) {
    if (curWord == targetWord) {
        answer = min(answer, curCount);
        return;
    }

    for (int i = 0; i < (int)words.size(); ++i) {
        string nextWord = words[i];
        if (visited[i]) continue;
        if (isOnlyOneDiff(curWord, nextWord)) {
            visited[i] = true;
            dfs(nextWord, curCount + 1);
            visited[i] = false;
        }
    }
}

int solution(string b, string t, vector<string> ws) {
    targetWord = t;
    words = ws;

    dfs(b, 0);
    if (answer == INT_MAX) answer = 0;
    return answer;
}
```

---

## 피드백
* `INT_MAX` 는 `climits` 안에 있다.
