---
title: 프로그래머스 뉴스 클러스터링
date: 2020-07-09-17:00
categories:
- PS

tags:
- Programmers
- PS
- Problem Solve
- 구현
- 해싱

---

## Problem : [뉴스 클러스터링](https://programmers.co.kr/learn/courses/30/lessons/17677)
> 유형 : 해싱

---
 
 
 
## 문제 해석
* 자카드 유사도를 구하라.

## 문제 재해석
* 자카드 유사도는 `교집합 / 합집합` 이다.
* 교집합은, 기본적인 교집합에서 `중복을 허용하는것 까지 확장`된 것이다. 

* 즉 `A {1 1 1}` 과 `B { 1 }` 이 있다면, 
* `교집합은 { 1 }` , `합집합은 { 1 1 1 }` 이 된다.
* 대문자, 소문자를 구분 하지 않는다.
* 문자는 두개씩 끊어내나, 특수문자가 공백이 포함된 경우 해당 쌍은 버린다.
* 합집합이 0 이 되는 경우는 `1` 로 처리한다.

## 주의할점
* 공백에 대한 처리를 해주어야 한다.

## 해결 전략 
* 문자열의 길이는 최대 `1000` 이다.
* 해싱을 이용하여, 문자열의 등장 여부, 등장 횟수를 관리하였다.

## 설계, 구현
* 전부 `소문자`로 만들어준다.
* 잘라낸 문자열 중 알파벳이 아닌것이 있으면 버린다.

### 교집합
* `A`에도 있고 , `B`에도 있는것중, `적은 등장 횟수`를 고른다.
### 합집합
* `A`에도 있고, `B`에도 있다면, 그중 `큰 등장 횟수`을 고른다.
  * `A`에는 있으나 `B`에는 없으면 `A의 개수`를 더한다.
* `B`에도 있고, `A`에도 있는것은 **위에서 처리**했으므로 `무시한다`.
  * `B`에는 있으나 `A`에는 없으면 `B의 개수`를 더한다.

## 디버깅
* 공백과 특수문자를 제거한, 새로운 소문자 문자열을 만든뒤 처리하는 실수를 했다.
* 이 경우에는 `a!b`의 경우 `ab`라는 문자열이 있는것 으로 **잘못 계산**해 버렸다.

---

## 코드

```c++
#include <bits/stdc++.h>

using namespace std;

unordered_map<string, int> Amap, Bmap;

int GetUnion() {
    int cnt = 0;

    const auto ANoExist = Amap.end();
    const auto BNoExist = Bmap.end();

    for (auto A : Amap) {
        string AStr = A.first;
        int ACnt = A.second;
        if (Bmap.find(AStr) != BNoExist) cnt += max(ACnt, Bmap[AStr]);
        else cnt += ACnt;
    }

    for (auto B : Bmap) {
        string BStr = B.first;
        int BCnt = B.second;
        if (Amap.find(BStr) != ANoExist) continue;
        else cnt += BCnt;
    }

    return cnt;
}

int GetIntersection() {
    int cnt = 0;
    const auto NoExist = Bmap.end();
    for (auto A : Amap) {
        string AStr = A.first;
        int ACnt = A.second;
        if (Bmap.find(AStr) == NoExist) continue;
        else cnt += min(ACnt, Bmap[AStr]);
    }
    return cnt;
}

void SplitTwoChar(string str, bool flag) {
    for (int i = 0; i < (int)str.size() - 1; ++i) {
        if (!isalpha(str[i]) or !isalpha(str[i + 1])) continue;
        string splitedStr = str.substr(i, 2);
        if (flag) Amap[splitedStr]++;
        else Bmap[splitedStr]++;
    }
}

string ParsingStr(string str) {
    string ret = "";
    for (const char& ch : str) {
        ret += tolower(ch);
    }
    return ret;
}

int solution(string str1, string str2) {
    double JcardSimillar = 0.0;
    str1 = ParsingStr(str1);
    str2 = ParsingStr(str2);

    SplitTwoChar(str1, true);
    SplitTwoChar(str2, false);

    int intersectionCnt = GetIntersection();
    int UnionCnt = GetUnion();

    const int base = 65536;
    if (UnionCnt == 0) JcardSimillar = 1;
    else JcardSimillar = double(intersectionCnt) / double(UnionCnt);

    return JcardSimillar * base;
}

```

---

## 피드백
* 문제 조건이 굉장히 여러개인경우, `따로 정리`해놓자.
* 그리고 어느 함수에서 요구하는지도 대충 옆에 기입해두자.
* `웹 IDE 코딩`은 오타 방지를 위해, `복사 붙여 넣기`를 이용하자.
