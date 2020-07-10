---
title: 프로그래머스 프렌즈4블록
date: 2020-07-10-11:00
categories:
- PS

tags:
- Programmers
- PS
- Problem Solve
- 구현
- 큐
- 카카오

---

## Problem : [프렌즈4블록](https://programmers.co.kr/learn/courses/30/lessons/17679)
> 유형 : 구현

---
 
## 문제 해석
* 블록이 몇개나 지워질지 구하라.

## 문제 재해석
* 지워지는 조건은 같은 모양이 `2x2` 형태가 된 경우이다.
* 겹치면서 `2x2`가 되는 경우도 같이 처리해준다.
* 문자는 `A - Z` 까지 등장한다.
* 보드의 크기는 최대 `30` 이다.

## 해결 전략
* `2x2`가 되었다고 바로 지우면 안된다. `지워짐을 확인하는 배열을 추가로 만든다` 
* 지운 다음, 아래가 빈 경우 밑으로 내려 재정렬 시키는 작업이 필요하다. `큐를 이용한다.`

## 구현, 설계
* 모든 좌표를 한번 순회한다.
* 좌표 기준으로 3방면을 탐색한다.
* 전부다 같다면, 지워질곳으로 표시해둔다.

<br>

![블록 옮기기](/post_images/friendsfourblock.png)

* 지워질 곳으로 예정 된곳은 `#`으로 치환한다.
* 열을 기준으로 아래에서 위로 올라오며 탐색하는데, `#`이 아닌것은 큐에 넣는다.
* 열 순회가 끝났으면,  모자른 크기만큼 `#`을 큐에 채워넣는다.
* 큐에서 하나씩 뺴며 아래부터 채워나간다.

## 디버깅
* 방문한 곳을 `#`로 치환하는 작업을 뺴먹었엇다.

---

## 코드

```c++
#include <string>
#include <vector>
#include <queue>
#include <iostream>
#include <cstring>

using namespace std;

vector<string> board;
bool boom[31][31];
int n, m;

const int dx[] = {0, 0, 1, 1};
const int dy[] = {0, 1, 0, 1};

int BoomCntRebase() {
    int cnt = 0;

    for (int ii = 0; ii < m; ++ii) {
        queue<char> q;
        for (int i = n - 1; i >= 0; --i) {
            if (boom[i][ii]) {
                board[i][ii] = '#';
                cnt++;
            }
            if (board[i][ii] == '#') continue;
            q.push(board[i][ii]);
        }
        int qSize = q.size();

        for (int i = 0; i < n - qSize; ++i) {
            q.push('#');
        }

        for (int i = n - 1; i >= 0; --i) {
            board[i][ii] = q.front();
            q.pop();
        }
    }
    return cnt;
}

bool IsFourBoom(int x, int y) {

    char ch = board[x][y];
    for (int dir = 1; dir < 4; ++dir) {
        int nx = x + dx[dir];
        int ny = y + dy[dir];

        if (nx < 0 or nx >= n or ny < 0 or ny >= m) return false;
        if (board[nx][ny] != ch) return false;
    }
    
    for (int dir = 0; dir < 4; ++dir) {
        int nx = x + dx[dir];
        int ny = y + dy[dir];

        if (boom[nx][ny]) continue;
        boom[nx][ny] = true;
    }

    return true;
}

bool IsBoom() {
    bool boomFlag = false;
    
    for (int i = 0; i < n; ++i) {
        for (int ii = 0; ii < m; ++ii) {
            if (board[i][ii] == '#') continue;
            if (IsFourBoom(i, ii)) boomFlag = true;
        }
    }

    return boomFlag;
}

void BoomReset() {
    memset(boom, false, sizeof(boom));
}

int solution(int mm, int nn, vector<string> b) {
    int answer = 0;
    n = mm;
    m = nn;
    board = b;

    while (1) {
        BoomReset();
        if (!IsBoom()) break;
        answer += BoomCntRebase();
    }

    return answer;
}
```

---
## 피드백
* 결과값이 오답이 나온다면, **코드부터 보는게 아니라**, `슈도코드`로 적어놓은걸 전부 다 구현 했는지 부터 확인하자.
