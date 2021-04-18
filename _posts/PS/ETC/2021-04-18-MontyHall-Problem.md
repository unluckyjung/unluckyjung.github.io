---
title: 몬티 홀 문제(Monty Hall Problem)를 Java, C++로 구현해보기
date: 2021-04-18-14:00
categories:
- PS

tags:
- 몬티 홀
- Cpp
- Java

photos: 
- https://user-images.githubusercontent.com/43930419/115135005-154b0d00-a050-11eb-9b56-fcab2e43d87e.png


---

## 몬티 홀 문제를 프로그래밍 언어로 풀어봅니다.
> 새로운 문을 선택할때 정말 승리확률이 66퍼가 나올까?

---

## 몬티홀 문제
> 염소를 뽑느냐, 스포츠카를 뽑느냐

- [몬티홀 문제](https://ko.wikipedia.org/wiki/%EB%AA%AC%ED%8B%B0_%ED%99%80_%EB%AC%B8%EC%A0%9C)는 조건부 확률과 베이즈 정리를 통해서, 선택을 바꾸는것이 당첨될 확률이 높아지는것이 증명된 문제이다.
- 필자의 경우 해당 문제를, 경시대회를 준비하던 초등학생때 수학이랑 관련된 무슨 책에서 처음 접했던 기억이 난다.
- 몬티홀 문제를 대충 요약하자면 아래와 같다.

```
세 개의 문 중에 하나를 선택하여 문 뒤에 있는 선물을 가질 수 있는 게임쇼에 참가했다. 
한 문 뒤에는 자동차가 있고, 나머지 두 문 뒤에는 염소가 있다. 
이때 어떤 사람이 예를 들어 1번 문을 선택했을 때, 게임쇼 진행자는 3번 문을 열어 문뒤에 염소가 있음을 보여주면서 1번 대신 2번을 선택하겠냐고 물었다. 
참가자가 자동차를 가지려할 때 원래 선택했던 번호를 바꾸는 것이 유리할까?

// 출처 : https://ko.wikipedia.org/wiki/%EB%AA%AC%ED%8B%B0_%ED%99%80_%EB%AC%B8%EC%A0%9C
```

- 일반적인 직관에 따르면, 선택을 바꾸지 않으나, 바꾸나 그 구간에서 1/2 을 선택하므로 `50%` 확률인 것처럼 보인다.
- 하지만, 선택을 바꾸지 않으면 확률은 `33%` 이고, 선택을 바꿀경우 확률은 `66%` 로 두배나 상승한다.
- 여기서 주목해서 봐야할 조건은 **사회자는 염소가 존재하는 문을 항상 연다는것이다.** **항상**
- 즉, 참가자가 첫선택에 자동차가 있는 문을 선택해도, **염소가 존재하는 2개의 문중 하나의 문을 무조건적으로 연다는 것이다**
- 이 문제에 대한 해설은 [위키](https://ko.wikipedia.org/wiki/%EB%AA%AC%ED%8B%B0_%ED%99%80_%EB%AC%B8%EC%A0%9C)에서 보도록하자.
- 이해가 안가도 괜찮다. 
- 당시 최고의 수학자인 [폴 에르되시](https://ko.wikipedia.org/wiki/%EC%97%90%EB%A5%B4%EB%90%98%EC%8B%9C_%ED%8C%94)도 통계적인 자료가 나오기 전까지는 66퍼 인것을 받아들이지 못했다고 한다.

---

## 프로그래밍을 해보자.
- 필자의 경우에도 66퍼인것을 받아들이기가 굉장히 힘들었던 기억이 있다.
- 하지만 이제는 프로그래밍 역량이 전혀 없던 초등학생이 아니므로, 한번 프로그래밍으로 문제를 구현하여 정말로 확률이 저렇게 달라지는지 확인해보자.

> 객체지향적으로 짜보려다가, 그냥 나이브하게 작동만을 위한 구현을 해보았다.


### JAVA

```java
package domain;

import java.util.Random;

class MontyHallProblemApp {
    public static void main(String[] args) {
        MontyHallProblem montyHallProblem = new MontyHallProblem(10000);
        montyHallProblem.run();

        System.out.println("stayWinRate  : " + montyHallProblem.getStayWinRate());
        System.out.println("changeWinRate : " + montyHallProblem.getChangeWinRate());
    }
}

public final class MontyHallProblem {
    private static final Random RANDOM = new Random();
    public static final int BOUND_MAX = 3;

    private int stayWinCount = 0;
    private int changeWinCount = 0;
    private final int tryCount;

    public MontyHallProblem(final int tryCount) {
        this.tryCount = tryCount;
    }

    public void run() {
        for (int i = 0; i < tryCount; ++i) {
            play();
        }
    }

    private void play() {
        int answer = getAnswerNumber();
        int firstNumber = selectFirstNumber();
        int openNumber = getOpenNumber(answer, firstNumber);

        if (winWhenStay(answer, firstNumber)) {
            stayWinCount++;
        }

        int secondNumber = selectChangeNumber(firstNumber, openNumber);
        if (winWhenChange(answer, secondNumber)) {
            changeWinCount++;
        }
    }

    private int getAnswerNumber() {
        return RANDOM.nextInt(BOUND_MAX);
    }

    private int selectFirstNumber() {
        return RANDOM.nextInt(BOUND_MAX);
    }

    private int getOpenNumber(final int answer, final int firstSelectNumber) {
        int openNumber = RANDOM.nextInt(BOUND_MAX);
        while (openNumber == answer || openNumber == firstSelectNumber) {
            openNumber = RANDOM.nextInt(BOUND_MAX);
        }
        return openNumber;
    }

    private boolean winWhenStay(final int answer, final int selectFirstNumber) {
        return answer == selectFirstNumber;
    }

    private boolean winWhenChange(final int answer, final int secondNumber) {
        return secondNumber == answer;
    }

    private int selectChangeNumber(final int firstSelectNumber, final int openNumber) {
        for (int secondNumber = 0; true; ++secondNumber) {
            if (secondNumber == firstSelectNumber || secondNumber == openNumber) continue;
            return secondNumber;
        }
    }

    public double getChangeWinRate() {
        return changeWinCount / (double)tryCount * 100.0;
    }

    public double getStayWinRate() {
        return stayWinCount / (double)tryCount * 100.0;
    }
}


```


```
> Task :MontyHallProblemApp.main()
stayWinRate  : 33.129999999999995
changeWinRate : 66.86999999999999
```

- 1만번의 수행을 해보았다.
- 선택을 바꾸는 경우에는 `66%` 에 근접한 결과가 나왔다. 
- 선택을 바꾸지 않는 경우에는`33%`라는 결과가 나왔다.


### c++

```c++
#include <iostream>
#include <random>

using namespace std;

int stayWinCount;
int changeWinCount;
int tryCount = 10000;

void montyHallProblem();

int getRandomNum(const int &start, const int &end);

int selectFirstNumber();

int getOpenNumber(const int &answer, const int &firstNumber);

int selectChangeNumber(const int &firstNumber, const int &openNumber);

int main() {
    for (int i = 0; i < tryCount; ++i) {
        montyHallProblem();
    }

    cout << "stayWinRate : ";
    cout << stayWinCount / double(tryCount) * 100.0<< "\n";
    cout << "changeWinRate : ";
    cout << changeWinCount / double(tryCount) * 100.0<< "\n";
}

void montyHallProblem() {
    int answer = getRandomNum(0, 2);
    int firstNumber = selectFirstNumber();
    int openNumber = getOpenNumber(answer, firstNumber);

    if (answer == firstNumber) {
        stayWinCount++;
    }

    int secondNumber = selectChangeNumber(firstNumber, openNumber);
    if (answer == secondNumber) {
        changeWinCount++;
    }
}

int selectFirstNumber() {
    return getRandomNum(0, 2);
}

int getOpenNumber(const int &answer, const int &firstNumber) {
    int openNumber = getRandomNum(0, 2);
    while (openNumber == answer || openNumber == firstNumber) {
        openNumber = getRandomNum(0, 2);
    }
    return openNumber;
}

int selectChangeNumber(const int &firstNumber, const int &openNumber) {
    for (int secondNumber = 0; true; ++secondNumber) {
        if (secondNumber == firstNumber || secondNumber == openNumber) continue;
        return secondNumber;
    }
}

// 랜덤 숫자 생성기
//end is included
int getRandomNum(const int &start, const int &end) {
    random_device rd;
    mt19937 gen(rd());
    uniform_int_distribution<int> dis(start, end);

    return dis(gen);
}
```

```
stayWinRate : 33.67
changeWinRate : 66.33
```

- class를 쓰지 않고, C 스타일로 나이브하게 작성해 보았다.
- 언어만 다를뿐 로직은 완전히 동일하다.
- 랜덤값을 만드는 부분에서 `srand()`를 사용하지 않았다.
  - 이 이유는 [이전 포스팅](https://unluckyjung.github.io/cpp/2020/05/04/Rand_func/)의 링크를 걸며 대신한다.


---

## 결론
> 선택을 바꾸는것이 승리 확률이 두배나 높다!

- 프로그램에 문제가 없다면, 귀납적으로 결론을 내는데 성공했다.
- 이해가 안갔어도 **이제는 받아들이자**

---

> 여담으로 이런 문제해결을 하기 위해 프로그래밍을 하는것에 엄청 재미를 느끼는데, 이 글을 쓰면서도 내내 즐거웠다.  
> 10분내 짧게 쓰려고 했던 글이지만 벌써 40분이 지났네..