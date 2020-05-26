---
title: Leetcode Challenge week1 [May]
date: 2020-05-26-16:00
categories:
- PS

tags:
- LeetCode
- PS
- Problem Solve

---

## [Leetcode Week 1 : May](https://leetcode.com/explore/challenge/card/may-leetcoding-challenge/534/week-1-may-1st-may-7th/)
> 1주차 문제 풀이

---


## Leetcode 챌린지
> 총 5주차중 2주차까지 밖에 진행하지 못한 챌린지.

* 시간적 여유의 부족이라고 적고, 핑계라고 읽습니다.
* 못푼 문제들은 6월중에 완주하는것이 목표입니다.
* 1주차 문제 풀이한것을 포스팅해봅니다.

---

## Day1 First Bad Version

### 문제 해석
* 불량품이 생기기 시작하는 첫 번째 버전을 찾아라.

### 문제 재해석
* 불량품을 만드는 가장 작은 버전을 찾아라
* 불량품이면 true를 리턴 하는 API를 최소한으로 호출하라.
* 탐색 문제이다.

### 해결 전략
* 문제에서 불량품은 항상 존재한다.
* 제품의 버전은 1번부터 N번 까지 있다.
* 정렬되어 있으므로, 이분 탐색을 통해 불량품을 찾는다.

### 설계, 구현
* mid가 불량품인 경우, 해당 버전을 끝으로 하고 재 탐색한다.
* mid가 불량품이 아닌 경우, 다음 버전을 시작으로 하고 재 탐색한다.
* `s + e` 연산에서 일어 날 수 있는 오버 플로를 방지하기 위해 
    * `(s + e) / 2` 대신 `s + (e - s) / 2` 로 mid 값을 찾았다.

### 첫 풀이 후 수정한 부분
* 재 탐색할 때, 답이 될 수 있는 것을 범위에 둘 필요가 없을 것 같아 수정한다.
* 답이 될 수 있는 버전은 `answer`에 저장해두고, 더 낮은 버전을 찾아 `e = mid - 1` 로 탐색해 나간다.

* 불량품이 된 버전이 나온 경우, 다음 탐색 범위의 끝은 항상 이전에 나온 불량품보다 낮은 버전을 탐색하므로, `answer`의 대소 비교가 불필요하다.

### 주의할 점
* Overflow를 조심하라.

### 디버깅
* `mid`가 불량품인 경우. 불량품인 mid를 포함해서 다시 재 탐색을 해야 하는데, 일반적인 원하는 수를 찾는 `e = mid -1` 로 잘못 구현하여 잘못된 값이 나왔었다.
* `s + e`를 하여 오버플로가 발생했었다.


### 회고

* 문제에서 찾는 것이 무엇 인지를 명확히 하자.
* 이분탐색 과정에서 오버플로를 방지하기 위해 `long long`을 이용했었는데, 새로운것을 배웠다.

```c++
// The API isBadVersion is defined for you.
// bool isBadVersion(int version);

#if 01
class Solution {
public:
    int firstBadVersion(int n) {
        int s = 1, e = n;
        int mid, answer = -1;
        
        while(s <= e){       
            mid = s + (e - s) / 2;              

            if(isBadVersion(mid)) {
                answer = mid;
                e = mid - 1;
            }
            else s = mid + 1;
        }      

        return answer;
    }
};

#else
class Solution {
public:
    int firstBadVersion(int n) {
        int s = 1, e = n;
        int mid;
        
        while(s <= e){       
            if(s == e) return s;

            mid = s + (e - s) / 2;              

            if(isBadVersion(mid)) e = mid;
            else s = mid + 1;
        }      
        

        return -1;
    }
};

#endif
```

---


## Day2 Jewels and Stones

 
### 문제 해석
* 가지고 있는 돌 중에서 돌의 개수를 찾아라

### 문제 재해석

* 내가 가지고 있는 것이 돌 인지를 확인하는 탐색 문제이다.

### 해결전략

#### 1번
* 내가 가진 돌을 하나씩 전부 보석 목록과 비교해본다.

#### 2번 
* 중복을 제거하고, 가진 돌의 목록을 만든다.
* 보석에 해당하는 것이, 내가 가진 돌 목록 중에 있는지 확인한다

### 설계 구현

* `count` 함수를 이용하여 탐색한다
* `unordered_set`을 이용하여 중복된 것을 제거하고 목록을 만든다.

### 피드백

* `set<char>s {string.begin(), string.end()` 이런 식으로 set을 선언함과 동시에, `iterator` 범위로 데이터를 삽입할 수 있음을 배웠다.

* 라인이 짧은 것을 원한다면 `s.count(jewel)`을 통한 구현도 가능하다.

* `unordered_map`을 이용한 풀이도 가능하다.

```c++
#if 0
class Solution {
public:
    int numJewelsInStones(string J, string S) {
        int sum = 0;
        
        for(char my_stone : J){
            
            sum += count(S.begin(), S.end(), my_stone);
        }
        
        return sum;
    }
};

#else

class Solution {
public:
    int numJewelsInStones(string J, string S) {
        
        unordered_set<char>s {J.begin(), J.end()};
        const auto no_exist = s.end();
        
        int cnt = 0;
        for(const char &jewel : S){
            if(s.find(jewel) != no_exist) cnt++;
        }
        
        return cnt;
    }
};

#endif
```


---

## Day3 Ransom Note

 
### 문제 해석
* magazine에 있는 단어들을 이용하여 ransomNote 문자열을 만들 수 있는지를 확인하라

### 문제 재해석
* 단순 구현 문제이다.

### 해결 전략
* 각 문자열마다 소문자들의 등장 횟수를 카운팅 하고 비교한다.

### 설계, 구현
* map을 이용하거나 소문자 영단어 라는 조건이 있으니 배열을 이용한다.
* magazine에서 등장한 소문자들의 숫자를 세어 놓는다.
* ransomNote를 이루는 소문자 각각의 등장 횟수가 magazine를 이루는 소문자 각각의 등장 횟수보다 작은지 확인한다.

```c++
#if 0
class Solution {
public:
    bool canConstruct(string ransomNote, string magazine) {
        unordered_map<char, int> m;
        
        for(char ch : magazine){
            m[ch]++;
        }
        
        for(char ch : ransomNote){
            if(m[ch] == 0) return false;
            else m[ch]--;
        }
        return true;
        
    }
};

#else

class Solution {
public:
	bool canConstruct(string ransomNote, string magazine) {
		int alpa_cnt[27] = { 0, };

		if ((int)ransomNote.size() > (int)magazine.size()) return false;

		for (char ch : magazine) {
			alpa_cnt[ch - 'a']++;
		}

		for (char ch : ransomNote) {
			if (--alpa_cnt[ch - 'a'] < 0) return false;
		}
		return true;

	}
};
#endif
```

---

## Day4 Number Complement

 
### 문제 해석
* 이진수로 만들고, 보수를 취한 결과 값을 리턴 하라.

### 문제 재해석
* 이진수로 만드는 문제이다.

### 해결 전략
* 2진수를 만들어보고, 각 자릿수를 확인한다.

### 구현, 설계
* `0`이 될 때까지 `2`로 나눈다.
* 보수를 취했을 때 `1` 이 되어야 하므로, `2`진수로 만든 자릿수가 `0`일때, 위치 만큼 `2`를 제곱 한 수를 더해준다.

```c++
class Solution {
public:
    int findComplement(int num) {
        int cnt = 0, ret = 0;
        
        while(num){
            if(num % 2 == 0) ret += pow(2, cnt);
            cnt++;  num /= 2;
        }
        return ret;
    }
};
```

---

## Day5 First Unique Character in a String

### 문제 해석
* 중복되어 등장 하지 않는 첫번째 알파벳을 찾는 문제이다.

### 문제 재해석
* 구현 문제이다.

### 해결 전략
* 각 알파벳이 등장한 횟수를 센다.

### 설계, 구현
* 한번 순회하며 몇 번 등장하는지 세어둔다
* 다시 한번 순회하며 1번 등장한 알파벳을 찾는다.

### 피드백
* 릿코드 문제가 아직 초반부라 너무 쉽다.
* 영어 해석을 잘하자.

```c++
class Solution {
public:
    
    int firstUniqChar(string s) {
        int repeat[27] = {0, };
        
        for(char ch : s){
            repeat[ch-'a']++;
        }
        
        int len = s.size();
        for(int i = 0 ; i < len; ++i){
            if(repeat[s[i] - 'a'] == 1) return i;
        }
        return -1;
        
    }
};
```

---

## Day6 Majority Element

 
### 문제 해석
* 과반수로 등장하는 요소를 찾아라.

### 문제 재해석
* 배열은 항상 차있다.
* 항상 과반수 요소는 존재한다.
* 과반수 요소를 알기 위해서는 각 요소가 몇번 등장했는지 알면 된다.

### 해결 전략, 설계, 구현

#### 1. 정렬
* 각 요소가 몇번 등장했는지 전부 기록한다.
* `vecotr`에 옮기고 `value` 를 기준으로 정렬한다.
* 가장 많이 등장한 요소를 찾는다

#### 2. 과반수 체크
* 과반수 이상 등장 했는지 유무만 따진다.
* 접근 하는 요소가 지금까지 과반수 이상 등장했는지 **먼저 체크한다.**
* 홀수인 경우를 주의해야 한다. 
* `{3, 2, 2}` 인 경우 `3/2 = 1` 이 되어 3인 경우도 답으로 처리 할 수 있다.
* 이를 방지하기 위해, 이번에 등장한 횟수가 `1/2`가 아닌 과반수가 넘을 경우에만 답으로 처리한다.

#### 3. 무어의 투표 알고리즘
* `과반수가 있음이 보장`될 경우에만 작동한다.
* 투표 수가 0이라면, 새로운 요소가 답 후보가 된다.
* 다음에 등장 하는 것이 다른 요소이면 투표수를 줄인다.
* 다음에 등장 하는 것이 같은 요소이면 투표수를 늘린다.
* 과반수로 등장하는 요소가 반드시 존재한다면, 그 요소는 항상 승자가 될 수 있다. 

### 피드백
* 무어의 투표 알고리즘에 대해서 새롭게 배웠다.
* `map`은 `key`값으로 정렬 되는 것을 잊지 말자

```c++
#include <bits/stdc++.h>
using namespace std;

#if 00
#define pp pair<int,int>
//Sort
class Solution {
public:
    static bool cmp(const pp& a, const pp& b) {
        if (a.second == b.second) return a.first < b.first;
        return a.second < b.second;
    }

    int majorityElement(vector<int>& nums) {
        unordered_map<int, int> m;
        for (const int& it : nums) {
            m[it]++;
        }

        vector<pp> vec(m.begin(), m.end());
        sort(vec.begin(), vec.end(), cmp);
        return vec.rbegin()->first;
    }
};

#elif 00 
// Major_check
class Solution {
public:
    int majorityElement(vector<int>& nums) {
        unordered_map<int, int> m;
        const int major_pivot = nums.size() / 2;

        for (const int& it : nums) {
            if (m[it]++ == major_pivot) return it;
        }
        return -1;
    }
};

# else
//Moore Voting
class Solution {
public:
    int majorityElement(vector<int>& nums) {
        int vote = 0; int answer = 0;
        
        for (const int& it : nums) {
            if (vote == 0) answer = it;

            if (it == answer) vote++;
            else vote--;
        }
        return answer;
    }
};

#endif

int main()
{
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);  cout.tie(NULL);
    freopen("input.txt", "r", stdin);

	vector<int> vec{ 3,3,4,4 };

    Solution answer;
	cout << answer.majorityElement(vec);

    return 0;
}
```

---


## Day7 Cousins in Binary Tree

 
### 문제 해석
* 두 원소가 사촌 관계인지 확인하라

### 문제 재해석
* 사촌 관계는 깊이가 같고, 부모가 다른 것이다.
* 노드는 최대 100개 밖에 없다.
* 트리를 순회 하면 된다. 
* DFS로 순회 시켰다.

### 해결 전략
* 루트 노트부터 트리를 순회하기 시작한다.
* 간 곳이 존재하는 노드라면 자신의 부모, 그리고 깊이를 저장한다.
* 모든 순회를 끝낸 다음 사촌 인지를 확인한다.

### 구현, 설계
* 루트 노드의 경우, 현재 깊이를 0, 부모를 -1로 잡고 시작했다.
* 가려는 곳이 NULL 이면 노드가 없는 것이다.
* DFS 를 이용해 모든 노드를 순회 했다.

### 피드백
* 아직도 그래프 문제에 익숙하지 않다.
* 좀 더 많은 문제를 통해 연습해야겠다.

```c++
#define pp pair<int,int>
#define depth first
#define parent second

pp info[102];

class Solution {
public:
    
    void dfs(TreeNode* node, int cur_depth, int cur_parent){
        if(node == NULL) return;
        
        info[node->val] = {cur_depth, cur_parent};
        dfs(node->left, cur_depth + 1, node->val);
        dfs(node->right, cur_depth + 1, node->val);
    }
    
    bool isCousins(TreeNode* root, int x, int y) {
        dfs(root, 0, -1);
        if(info[x].depth != info[y].depth) return false;
        if(info[x].parent == info[y].parent) return false;
        return true;
    }
};
```