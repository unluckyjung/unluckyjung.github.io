---
title: 프로그래머스 디스크 컨트롤러 Cpp, Java
date: 2020-06-03-16:00
categories:
- PS

tags:
- Programmers
- PS
- Problem Solve
- heap

---

## Problem : [디스크 컨트롤러](https://programmers.co.kr/learn/courses/30/lessons/42627)
> 유형 : 힙

---
 
 
 
## 해결전략
* 작업 요청으로부터 종료까지 걸린시간의 평균을 최소화 하라.

## 문제 재해석
* `jobs` 은 최대 500개 이다.
* 스케줄링 알고리즘 중 하나인 `SJF` 알고리즘이다.

## 해결 전략
* 작업을 진행 하는 동안, 들어온 작업들 중 가장 빨리 끝낼 수 있는 작업을 처리한다.
* `SJF`를 구현한다.

## 설계, 구현
* `우선순위 큐`를 이용해서, 시작 되는 순서를 정했다.
* `우선순위 큐`를 이용해서, 가장 빨리 끝낼 수 있는 작업을 구했다.
* 요청 되는 시점 순으로 관리한다.
* 작업이 종료된 후 다음 작업을 고를때, 요청된 시간을 보고 , 작업 시간 기준으로 이전에 들어온 작업들을 `작업큐`에 넣었다.
* 한 작업이 끝날때 마다 시간이 변하므로 계속 확인해준다.

---


## 주의할 점
* 작업큐의 모든 과정을 끝냈지만, 이전에 아무런 요청이 들어오지 않은 경우를 처리 해줘야 한다.
* `3초`에 모든 작업이 끝. 다음 작업은 `5초`에 요청 되어짐.
* 이런 경우를 처리할 수 있어야 한다.

## 디버깅
* `first` `second`를 잘못 뒤섞어 썼었다.
* `while`문 종료 조건을 잘못 주었다.

---

## 코드

### cpp

```c++

#include <bits/stdc++.h>
#define pp pair<int,int>

using namespace std;

priority_queue<pp, vector<pp>, greater<pp>> input_pq, job_pq;
int answer;
int cur_time;

void Select_Job() {
    auto job = job_pq.top();    job_pq.pop();

	if (job.second > cur_time) cur_time = job.second;

	int fin_time = cur_time + job.first;
	answer += (fin_time - job.second);
	cur_time = fin_time;
}

void Pq_Check() {
    if (job_pq.empty()) {
        auto job = input_pq.top(); input_pq.pop();
        job_pq.push({ job.second, job.first });
    }

    while (!input_pq.empty() and input_pq.top().first <= cur_time) {
        auto job = input_pq.top(); input_pq.pop();
        job_pq.push({ job.second, job.first });
    }
}

int solution(vector<vector<int>> jobs) {
    for (auto vec : jobs) {
        input_pq.push({ vec[0], vec[1] });
    }

    while (!input_pq.empty() or !job_pq.empty()) {
        Pq_Check();
        Select_Job();
    }

    answer /= jobs.size();
    return answer;
}

int main() {

    vector<vector<int>> jobs{ {0, 3} ,{1, 9},{2, 6} };
    cout << solution(jobs);
    return 0;
}
```

### java

```java
import java.util.PriorityQueue;

class Solution {
    int readyTime;
    int curTime;
    PriorityQueue<Job> jobPq = new PriorityQueue<>();
    PriorityQueue<Ready> readyPq = new PriorityQueue<>();

    public int solution(int[][] jobs) {
        int jobCount = jobs.length;
        for (int[] job : jobs) {
            jobPq.add(new Job(job[0], job[1]));
        }

        while (!(jobPq.isEmpty() && readyPq.isEmpty())) {
            work();
            jobQMoveToReadyQ();
        }

        return readyTime / jobCount;
    }

    private void work() {
        if (readyPq.isEmpty()) {
            Job selectJob = jobPq.remove();
            curTime = selectJob.callTime + selectJob.needTime;
            readyTime += curTime - selectJob.callTime;
        } else {
            Ready selectJob = readyPq.remove();
            curTime += selectJob.needTime;
            readyTime += curTime - selectJob.callTime;
        }
    }

    private void jobQMoveToReadyQ() {
        while (!jobPq.isEmpty() && jobPq.element().callTime <= curTime) {
            readyPq.add(new Ready(jobPq.element().needTime, jobPq.element().callTime));
            jobPq.remove();
        }
    }
}

class Job implements Comparable<Job> {
    int callTime;
    int needTime;

    public Job(final int callTime, final int needTime) {
        this.callTime = callTime;
        this.needTime = needTime;
    }

    @Override
    public int compareTo(final Job o) {
        if (this.callTime == o.callTime) return Integer.compare(this.needTime, o.needTime);
        return Integer.compare(this.callTime, o.callTime);
    }
}

class Ready implements Comparable<Ready> {
    int needTime;
    int callTime;

    public Ready(final int needTime, final int callTime) {
        this.needTime = needTime;
        this.callTime = callTime;
    }

    @Override
    public int compareTo(final Ready o) {
        if (this.needTime == o.needTime) return Integer.compare(this.callTime, o.callTime);
        return Integer.compare(this.needTime, o.needTime);
    }
}
```

---

## 피드백
* 처음에는 `naive`하게 그냥 시간을 순차적으로 진행 시킬까 했는데 시간 복잡도를 고려해보니 `5억`이 나와서, 좀 더 고민하고 구현 했다. 이점은 잘한듯
* `pair`의 앞 뒤를 바꿔야 하는 경우, 매크로 함수를 사용하지 말자
* 탈출 조건, 종료 조건은 항상 고민을 한번만 더 해보자.