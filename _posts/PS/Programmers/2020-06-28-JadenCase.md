---
title: 프로그래머스 JadenCase 문자열 만들기
date: 2020-06-28-22:00
categories:
- PS

tags:
- Programmers
- PS
- Problem Solve
- 문자열

---

## Problem : [JadenCase 문자열 만들기](https://programmers.co.kr/learn/courses/30/lessons/12951)
> 유형 : 문자열 처리

---
 
 
## 문제 해석
* 주어진 문자열을 `JadenCase` 문자열로 치환하여라

## 문제 재해석 
* 문자열별 구분은 공백으로 한다.
* 맨 앞글자만 대문자.
* 나머지는 소문자로 변경한것이 `JadenCase` 이다.

## 해결 전략
* 바로 앞이 `공백인 경우`, 해당 문자는 대문자로 바꾼다.
* 바로 앞이 `공백이 아닌 경우`, 어느 문자의 뒤에 붙어 있다는 이야기 이므로, 맨 앞글자가 아니게 된다.
* 이 경우는 전부 소문자로 처리 해주면 된다.

## 주의할 점
* 공백이 연속적으로 여러개가 나올 수 있다.

---

## 코드

```c++
#include<bits/stdc++.h>

using namespace std;

string solution(string s) {
    string answer = "";

    for (int i = 0; i < s.size(); ++i) {
        if (i == 0) answer += toupper(s[i]);
        else {
            char before = s[i - 1];
            if(before == ' ') answer += toupper(s[i]);
            else answer += tolower(s[i]);
        }
    }

    return answer;
}
```

---

## 피드백
* 당연하게 생각되는것이 예외일 수 있다.
    * 이 부분을 놓쳐서 `split` 함수를 구현하고, 공백을 기준으로 나눈뒤 `JadenCase`로 만들어 제출하고 틀렸었다.
* `tolower`, `toupper`은 인자가 알파벳이 아닌 경우,  **에러가 나는 것이 아닌**, 원본이 리턴 된다.