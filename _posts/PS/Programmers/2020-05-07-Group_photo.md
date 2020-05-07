---
title: 프로그래머스 단체사진 찍기
date: 2020-05-07-22:00
categories:
- PS

tags:
- baekjoon
- PS
- Problem Solve
- 완전탐색
- 카카오코드 본선

photos:
- https://t1.kakaocdn.net/codefestival/picture.png

---

## Problem : [단체사진 찍기](https://programmers.co.kr/learn/courses/30/lessons/1835)
> 유형 : 완전탐색

---
 
## 문제 해석
* 조건에 만족하며 줄을 서는 경우의 수를 구하라
* 총 8명이 있다.

## 문제 재해석 (7)
* 모든 경우를 다 만들어보고 조건에 맞는지 확인 해본다.
* **완전 탐색** 문제이다.

## 해결 전략 (7)
* 모든 경우를 다 만들어본다.
* 만들어본 경우에서 원하는 조건에 해당하는지 확인해본다.
* 모든 조건을 통과하면 경우의 수를 증가 시킨다.

## 설계, 구현 (14)
* `permutation`으로 모든 조합을 만들어 본다.
* `a인원`의 위치를 찾고, `b인원`의 위치를 찾는다.
* 둘의 거리 차이를 구한다.  
* find의 경우, 위치 인덱스를 리턴 하기 때문에 `-1`를 한번 더 해준다.

---

## 코드

```c++
#include <bits/stdc++.h>

using namespace std;

// 전역 변수를 정의할 경우 함수 내에 초기화 코드를 꼭 작성해주세요.

string friends = "ACFJMNRT";


bool Is_Can(char cmd, int dist, int want_dist) {
	if (cmd == '=') return dist == want_dist;
	else if (cmd == '<') return dist < want_dist;
	else if (cmd == '>') return dist > want_dist;
}

int solution(int n, vector<string> data) {
    int answer = 0;

    do {
        int is_can = true;
        for (string d : data) {
            char a = d[0], b = d[2];
            char cmd = d[3];
            int want_dist = d[4] -'0';

            int dist = friends.find(a) - friends.find(b);
            dist = abs(dist) - 1;

            if(Is_Can(cmd, dist, want_dist)) continue;
            is_can = false;
            break;
        }
        if (is_can) answer++;

    } while (next_permutation(friends.begin(), friends.end()));

    return answer;
}

int main() {

    vector<string> data{ "N~F=0", "R~T>2" };

    cout << solution(2, data);

    return 0;
}
```


---


## 피드백
* 없음