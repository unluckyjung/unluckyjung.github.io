---
title: 프로그래머스 오픈 채팅방 / C++
date: 2020-07-10-17:00
categories:
- PS

tags:
- Programmers
- PS
- Problem Solve
- 해싱
- 문자열
- 카카오

---

## Problem : [오픈 채팅방](https://programmers.co.kr/learn/courses/30/lessons/42888)
> 유형 : 해싱

---
 
## 문제 해석
* 마지막에 보게 되는 메시지를 리턴하라.

## 문제 재해석
* 유저의 구별은 `uid`로 한다.
* 유저의 닉네임은 변경 될 수 있다.
* 유저의 닉네임이 변경 된 경우, 채팅방 메시지에 남긴 로그도 변경 된다.

## 해결 전략
* `uid`를 기준으로 `닉네임`을 기억한다.
* `최종적으로 변한 닉네임`에만 관심이 있다.
* 닉네임의 변화를 관찰하고, 최종적으로 변경된 닉네임만 기억한다.

## 설계, 구현
* `해싱`을 이용하여 닉네임을 기억한다.
* 한번 순회하며, 닉네임이 변경되는 `Enter` 와 `Change` 의 경우에만 관심을 가진다.
* 다시 한번 순회하며, 로그 메시지를 남기는 `Enter`와 `Leave`의 경우에만 관심을 가진다.

---

## 코드

```c++
#include <bits/stdc++.h>

using namespace std;

unordered_map<string, string> id;
vector<string> answer;

void GetResult(string record) {
    string result = "";

    stringstream ss(record);
    string cmd, uid, nickName;

    ss >> cmd;
    if (cmd == "Change") return;

    ss >> uid;
    result += id[uid] + "님이 ";
    if (cmd == "Enter") result += "들어왔습니다.";
    else if (cmd == "Leave") result += "나갔습니다.";

    answer.push_back(result);
}

void GetRealNickName(string record) {
    stringstream ss(record);
    string cmd, uid, nickName;

	ss >> cmd;

	if (cmd == "Leave") return;

	ss >> uid >> nickName;
	id[uid] = nickName;
}

vector<string> solution(vector<string> records) {

    for (string record : records) {
        GetRealNickName(record);
    }

    for (string record : records) {
        GetResult(record);
    }

    return answer;
}
```

---

## 피드백
* `sstream`은 `stringstream` 안에 있다.
* `bits/stdc++` 사용을 좀 줄여야겠다.
* 검색 노출을 위해서, 앞으로는 사용 언어도 뒤에 붙일 예정이다.