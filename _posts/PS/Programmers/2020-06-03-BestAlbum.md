---
title: 프로그래머스 베스트 앨범
date: 2020-06-03-12:51
categories:
- PS

tags:
- Programmers
- PS
- Problem Solve
- 해싱

---

## Problem : [베스트 앨범](https://programmers.co.kr/learn/courses/30/lessons/42579)
> 유형 : 해싱

---
 
 
## 문제 해석
* 장르별에서 가장 많이 재생된 노래 두개씩을 뽑아라.

## 문제 재해석
* 가장 많이 재생된 장르를 고른다.
* 장르 내에서 가장 많이 재생된 노래를 고른다.
* 장르 내에서 가장 많이 재생된 노래가 여러 개라면, 낮은 고유 번호의 노래를 고른다.

## 해결 전략
* 장르가 뭐가 나올지 모른다. `해싱`을 이용한다.
* 노래 개수는 1만개가 넘지 않는다. 장르 종류도 100개 이하이다. 
* `모든 장르는 재생된 횟수가 다르다` 는 점을 캐치 한다.
* 장르 이름에 따른 노래들을 정리한다.
* 

## 구현, 설계
* 장르별로 노래가 몇 번 재생 되었는지 기록한다.
* 해당 장르에 속한 노래의 재생 시간, 고유 번호를 기록한다.
* 조건에 맞춰서 정렬해 나가며 답을 낸다.

* `map`은 오름차순 정렬이다. 이것을 `음수`로 취해 내림차순 정렬로 유도할 수 있다.
* 첫 번째 풀이에서는 `cmp` 를 따로 만들어서 정렬했다.
* 두 번째 풀이에서는 `음수` 를 이용해 내림차순을 유도해서 정렬했다.

---

## 주의할 점
* 장르에 속한 곡이 하나인 경우를 고려한다.

---

## 코드

```c++

#if 00
#include<bits/stdc++.h>
using namespace std;

struct music {
    int play_cnt;
    int music_number;
};

bool cmp(const music &a, const music &b) {
    if (a.play_cnt == b.play_cnt) return a.music_number < b.music_number;
    else return a.play_cnt > b.play_cnt;
}



map<string, pair<int, vector<music> > > musics;

vector<int> answer;
vector<int> solution(vector<string> genres, vector<int> plays) {
    int len = plays.size();

    for (int i = 0; i < len; ++i) {
        string type = genres[i];
        int type_cnt = plays[i];
        int uniq_num = i;

        musics[type].first += type_cnt;
        musics[type].second.push_back({ type_cnt, uniq_num });
    }

    map<int, vector<music> > tmp;

    for (auto it = musics.begin(); it != musics.end(); ++it) {
        tmp[it->second.first] = it->second.second;
    }
    
    for (auto it = tmp.rbegin(); it != tmp.rend(); ++it) {
        vector<music> &vec = (it->second);
        sort(vec.begin(), vec.end(), cmp);

        for (int i = 0; i < vec.size() && i < 2; ++i) {
            answer.push_back(vec[i].music_number);
        }
    }

    return answer;
}

#else

#include<bits/stdc++.h>
#define pp pair<int, int>

using namespace std;

unordered_map<string, pair<int, vector<pp> > > musics;

vector<int> answer;
vector<int> solution(vector<string> genres, vector<int> plays) {
    int len = plays.size();

    for (int i = 0; i < len; ++i) {
        string type = genres[i];
        int type_cnt = -plays[i];
        int uniq_num = i;

        musics[type].first += type_cnt;
        musics[type].second.push_back({ type_cnt, uniq_num });
    }

    map<int, vector<pp> > tmp;

    for (auto it = musics.begin(); it != musics.end(); ++it) {
        tmp[it->second.first] = it->second.second;
    }

    for (auto it = tmp.begin(); it != tmp.end(); ++it) {
        vector<pp>& vec = it->second;
        sort(vec.begin(), vec.end());

        for (int i = 0; i < vec.size() && i < 2; ++i) {
            answer.push_back(vec[i].second);
        }
    }
    return answer;
}

#endif

int main() {

    vector<string> ge = { "classic", "pop", "classic", "classic", "pop", "lock" };
    vector<int> p = { 500, 600, 150, 800, 2500, 100 };
    vector<int> ans = solution(ge, p);

    for (auto it : ans) {
        cout << it << " ";
    }

    return 0;
}
```


---

## 피드백
* `map`은 `key`값을 기준으로 정렬 된다는 것을 또 깜박 했다.
* 항상 양수만 나온다는 조건에서는 `음수`를 이용하여 의도적으로 내림차순을 유도할 수 있다는 것을 알게 되었다.