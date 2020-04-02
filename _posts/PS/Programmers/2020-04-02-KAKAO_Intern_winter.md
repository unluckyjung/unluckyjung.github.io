---
title: 카카오 2019 겨울 인턴십 코딩 테스트 문제풀이
date: 2020-04-02-22:00
categories:
- PS

tags:
- Programmers
- PS
- Problem Solve
- KAKAO

---

## 카카오 겨울 인턴십 문제풀이
> 2019년 11월에 응시된 문제가 프로그래머스에서 오픈되어 풀어봅니다.

---

### 첫 인턴십 문제 공개
> 이전까지는 공개를 하지 않았다.

* 카카오에서 처음으로 인터십 문제를 공개하였습니다.  
* 문제는 [프로그래머스](https://programmers.co.kr/learn/challenges) 에서 풀어보실 수 있습니다.  

### 글을 작성해도 될까?
> 공개되기 전까지는 당연히 불가능 했다.

* [다른 블로거분](https://blog.niceb5y.net/2019-kakao-developer-winter-internship-solution/) 을 보니 11월에 시험에 응시하신 후 바로 글을 작성하셨다가, `저작권 침해`라는 이유로 글을 내려달라는 권고를 받으셨었다.
* 하지만 이번에는 프로그래머스에서 공개적으로 오픈한 문제이다
* 혹시나 몰라서 찾아보았고 [링크](https://programmers.zendesk.com/hc/ko/articles/360034546572-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%A8%B8%EC%8A%A4%EC%9D%98-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%EB%AC%B8%EC%A0%9C-%ED%92%80%EC%9D%B4%EB%A5%BC-%EA%B0%9C%EC%9D%B8-%EB%B8%94%EB%A1%9C%EA%B7%B8-%EA%B9%83%ED%97%99-%EA%B8%B0%ED%83%80-%EC%82%AC%EC%9D%B4%ED%8A%B8%EC%97%90-%EC%98%AC%EB%A0%A4%EB%8F%84-%EB%90%98%EB%82%98%EC%9A%94-) 를 보니 공개된 문제는 게시해도 되는것 같다. 
	* **문제가 될시, `ybook2012@gmail.com` 이쪽으로 연락주시면 바로 게시글을 내리도록 하겠습니다.**

---

### 문제풀이
> 사실 제 풀이를 보는것보다 [기술블로그](https://tech.kakao.com/2020/04/01/2019-internship-test/)의 문제 해설을 보시는게 **2만배** 낫습니다.  
저는 알고수가 아니며, 이 글은 저의 기록을 통한 학습용으로 작성하는 용도입니다.

---

#### [1번](https://programmers.co.kr/learn/courses/30/lessons/64061)

> 구현, 스택, 큐

* 그냥 그대로 `구현`하면 되는 문제였습니다.
* 게임 화면의 상태를 입력 받을때 `큐`를 이용하여 라인별로 인형을 관리하였고
* 크레인으로 인형을 뽑아간것을 `스택`에 넣어보며 인형이 없어질 수 있음 여부를 확인했습니다.


##### 코드

```c++

#include<bits/stdc++.h>

using namespace std;

vector<vector<int>> board;
vector<int> moves;

queue<int> q[32];
stack<int> basket;

int answer;
int n;

void basket_check(int cur) {
    if (!basket.empty()) {
        if (basket.top() == cur) {
            basket.pop();
            answer += 2;
            return;
        }
    }
    basket.push(cur);
}

void pick() {
    for (auto i : moves){
        if (q[i].empty())continue;
        int cur = q[i].front(); q[i].pop();
        basket_check(cur);
    }
}

void refactory() {
    for (int ii = 0; ii < n; ++ii) {
        for (int i = 0; i < n; ++i) {
            int doll = board[i][ii];
            if (doll == 0)continue;
            q[ii+1].push(doll);
        }
    }
}

int solution(vector<vector<int>> b, vector<int> m) {
    board = b;
    moves = m;
    n = board.size();

    refactory();
    pick();

    return answer;
}

```

---


#### [2번](https://programmers.co.kr/learn/courses/30/lessons/64065)

> 문자열 처리

* `문자열에서 원하는 정보를 파싱` 할 수 있는지를 물어보는 문제였습니다.
* 처음에는 정규식을 이용해 풀까 하다가, 그냥 구현도 쉬워 보이길래 단순히 구현했습니다.
* 문제 해석에서 조금 고생을 했는데
	* 문자열 집합중 가장 **적은 개수의 원소를 가지고 있는 집합**부터 오름차순으로 확인 합니다.
	* 집합을 탐색하는데, **이전에 등장하지 않았던 숫자**는 튜플을 이루는 숫자가 됩니다.

* 문자열 파싱은
	* 맨 처음과 맨 끝의 괄호는 무시하고
	* `{` 을 발견시, 이후에 `}`이 등장하는 인덱스를 찾았습니다.
	* 그 사이를 탐색하는데 `,` 이 나오거나 `}` 이 나오면 문자열을 숫자로 변경하는 방법으로 구현했습니다.

##### 코드

```c++

#include <bits/stdc++.h>

using namespace std;

string str;
vector<int> answer;
vector<int> tuple_set[100002];
bool visit[100002];
int MAX;

void get_answer() {
    for (int i = 1; i <= MAX; ++i) {
        for (auto it : tuple_set[i]) {
            if (visit[it])continue;
            visit[it] = true;
            answer.push_back(it);
        }
    }
}

void get_number(int s, int e) {
    string n_string;
    vector<int> vec;
    for (int i = s; i <= e; ++i) {
        if (str[i] == ',' or str[i] == '}') {
            vec.push_back(stoi(n_string));
            n_string = "";
        }
        else n_string += str[i];
    }
    tuple_set[vec.size()] = vec;
    MAX = max(MAX, (int)vec.size());
}

void tuple_find() {
    for (int i = 1; i < (int)str.size() - 1; ++i) {
        if (str[i] != '{') continue;
        int e = str.find('}', i);
        if (e == -1)return;
        get_number(i + 1, e);
        i = e + 1;
    }
}


vector<int> solution(string s) {
    str = s;

    tuple_find();
    get_answer();

    return answer;
}

```

---


#### [3번](https://programmers.co.kr/learn/courses/30/lessons/64064)

> 완전탐색, 비트마스크

* 제일 구현하기 귀찮았고, 상대적으로 제일 오래 걸린 문제입니다.
* 그냥 모든 경우를 다 만들어보면됩니다.
* `비트마스크`와 `permutation` 을 이용하여, 응모자 아이디 목록에서 불량 사용자가 될 수 있는 모든 경우를 뽑아 보았습니다.
*  **이 부분은 제가 처음에 놓쳤던 부분인데,** 위에서 뽑은 경우를 불량 사용자와 매칭 시켜볼때
	* 저의 구현방식으로는 불량 사용자의 경우도 모두 만들어 비교해야 합니다.
	* 왜냐하면 `A 사용자`가 불량사용자 목록의 `a, b와 매칭되고` , `B 사용자`가 불량 사용자의 `a` 와 매칭될때
	* `A 사용자`가 `a`랑 매칭을 해버리면, `B 사용자`는 매칭 되는것이 없어버립니다.
	* **3번 테스트 케이스를 제공하지 않았으면, 오답을 제출했을것 같습니다**



##### 코드

```c++

#include <bits/stdc++.h>

using namespace std;

vector<int> combi;
vector<string> user_id, banned_id;

bool banned[10];

bool compare(string a, string b) {
    if (a.size() != b.size()) return false;
    for (int i = 0; i< a.size(); ++i) {
        if (b[i] == '*')continue;
        if (a[i] != b[i])return false;
    }
    return true;
}

bool ban_check(string candidate_id) {
    for (int i = 0; i < banned_id.size(); ++i) {
        if (banned[i])continue;
        if (compare(candidate_id, banned_id[i])) {
            banned[i] = true;
            return true;
        }
    }
    return false;
}


void Init() {
    for (int i = 0; i < (int)banned_id.size(); ++i) {
        combi.push_back(1);
    }

    for (int i = 0; i < user_id.size() - banned_id.size(); ++i) {
        combi.push_back(0);
    }

    sort(banned_id.begin(), banned_id.end());
}


int solution(vector<string> u, vector<string> b) {
    int answer = 0;

    user_id = u; banned_id = b;
    Init();
    do {
        vector<string>backup_ban_id = banned_id;

        do {
            int flag = true;
            memset(banned, false, sizeof(banned));

            for (int i = 0; i < combi.size(); ++i) {
                if (combi[i] == 0)continue;
                if (!ban_check(user_id[i])) {
                    flag = false;
                    break;
                }
            }
            if (flag) {
                answer++;
                break;
            }
        } while (next_permutation(banned_id.begin(), banned_id.end()));

        banned_id = backup_ban_id;

    } while (next_permutation(combi.rbegin(), combi.rend()));

    return answer;
}

```

---


#### [4번](https://programmers.co.kr/learn/courses/30/lessons/64063)

> Union Find

* **다른 방법이 있을 수 있습니다만** 제 머리로는 `Union Find`로 밖에 접근이 안됩니다.
* 방이 될 수 있는 숫자는 `10^12` 까지 있습니다
	* 미리 배열을 만들어둘 수 없는 크기입니다. 이를 해결하기 위해 `map`을 사용 했습니다.
	* 빈방의 여부도 `map`을 이용하여 확인 했습니다.
* 방의 번호를 하나씩 받아서 처리하며, 해당 방번호가 비었는지 확인해봅니다. 
	* 방이 비었다면, 방번호가 트리에서 `루트 노드`였다는 이야기 입니다.
	* 방이 비지 않았다면, 요구한 방번호의 `루트 노드`를 찾으면 됩니다.
* `유니온 파인드` 자료구조에 대한 설명은 [갓라이](https://kks227.blog.me/220791837179) 블로그에서 보시면 될것 같습니다.

##### 코드

```c++

#include <bits/stdc++.h>
#define ll long long

using namespace std;

unordered_map<ll, ll> room;
const auto empty_room = room.end();

ll _find(ll n) {
    if (room.find(n) == empty_room) return n;

    room[n] = _find(room[n]);
    return room[n];
}

vector<long long> solution(long long k, vector<long long> room_number) {
    vector<ll> answer;

    for (auto n : room_number) {
        ll get_room = _find(n);

        answer.push_back(get_room);
        room[get_room] = get_room + 1;
    }
    return answer;
}

```

---

#### [5번](https://programmers.co.kr/learn/courses/30/lessons/64062)

> 슬라이딩 윈도우, 해시

* 스톤의 원소들의 값은 `200,000,000` 이하입니다. 
	* 역시 미리 만들어 둘 수 없으므로 `map`을 이용했습니다.
	* 해당 스톤의 존재 여부도 `map`을 이용하여 확인했습니다.
* 구간의 길이를 유지하기위해서 `deque` 자료구조를 이용했습니다.
* 돌을 `k` 만큼 건너 뛸 수 있다는것은
	* `k길이의` 구간내 최대값을 구하면, 해당 구간을 통과 할 수 있는 인원의 최대값을 구할 수 있게 됩니다.
* 만약 `{ 1, 2, 3, 4 }` , `k = 2` 라면
	* `{1, 2}` 구간은 **2명**이 통과할 수 있습니다.
	* `{3, 4}` 구간은 **4명**이 통과할 수 있습니다.
	* 이때 `{1, 2}` 구간을 통과해야 `{3, 4}` 구간을 통과할 수 있습니다. 
		* 구간별 통과한 인원수중 **최솟값** 을 구하면됩니다

* 사실 제일 먼저 떠올랐던건 `세그먼트 트리` 입니다. 아마 세그먼트 트리로도 풀 수 있을것 같습니다.

##### 코드

```c++

#include <bits/stdc++.h>

using namespace std;

map<int, int> m;
deque<int> stone_list;

int find_max() {
    return m.rbegin()->first;
}

bool empty_check(int will_remove) {
    return --m[will_remove] == 0;
}

int solution(vector<int> stones, int k) {
    int answer = INT_MAX;

	for (int it : stones) {
		stone_list.push_back(it);  
		m[it]++;

		if (stone_list.size() < k)continue;
		if (stone_list.size() > k) {
			auto will_remove = stone_list.front();
			stone_list.pop_front();

			if (empty_check(will_remove)) m.erase(will_remove);
		}
		answer = min(answer, find_max());
	}
    return answer;
}

```

> 게시된 글에서는 들여쓰기가 자꾸 이상하게 적용 되네요. 이유를 모르겠습니다

---

### 피드백

* 완전탐색의 경우, 정말 모든 경우를 다 확인했는지 확인해보자 
* `rebegin()`을 `rend()`로 써놓는 실수를 했었다.