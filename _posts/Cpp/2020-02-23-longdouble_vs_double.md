---
title: long double 과 double 의 차이.
categories:
- Cpp

tags:
- Cpp
- gcc, vc++
- double
---

## long double vs double 
> 오차 범위 때문에, double로는 WA가 나오나, long double로는 AC가 나오는 [문제](https://www.acmicpc.net/problem/1575)를 우연찮게 보았다.  
> 사실 이전까지 long double 이라는 자료형은 double 형과 같은 크기라고 **잘못!** 알고 있었다.

---

### gcc와 vc++ 컴파일러 차이에 따라서 다르다.

* **결론부터 이야기하자면**
* 표준 문법을 따르는 `gcc, g++` 에서는 long double이 `16byte`
* `vc++` 에서는 long double이 `8byte` 로 크기가 찍힌다. *vc++은 VS에서 사용되는 컴파일러다*
	* vs를 이용하여 문법을 배웠던 나는, 표준에서 벗어나는 문법을 배웠고, 그렇게 알고 지내온 것이다.

### long double은 정밀도가 아주 높다.

* double  : `±2.23 x 10^-308` to `±1.80 x 10^308`
* long double  : `±3.36 x 10^-4932` to `±1.18 x 10^4932`
* 까지의 범위를 커버한다.

* [이 사이트를 참고했다.](https://www.learncpp.com/cpp-tutorial/floating-point-numbers/) *틀린게 있다면 지적 바랍니다.*

---


### 예시

```c++
#include <bits/stdc++.h>
#include<unordered_set>
using namespace std;

int main()
{
    ios_base::sync_with_stdio(false);
    //freopen("input.txt", "r", stdin);
    cin.tie(NULL);

    long double ld = 3.14L;
    double d = 3.14;

    cout << "long double size : " << sizeof(ld) << "\n";
    cout << "     double size : " << sizeof(d) << "\n";
    return 0;
}

```

* output in `g++ (mingw)`

```c++
 long double size : 16
 long double size : 8
```

* output in `vc++ (visual studio)`

```c
 long double size : 8
 long double size : 8
```

---

> 근시일내에 C, C++ 관련 원서를 한번 쭉 읽어 봐야겠다. *(VS로 환경설정부터 시작하는 국내 저서말고)*  
앞으로, 부동 소수점 정밀도가 많이 필요한 문제는 long double을 사용하자.

---

#### Reference

* [Learncpp](https://www.learncpp.com/cpp-tutorial/floating-point-numbers/)  
* [Stack Overflow](https://stackoverflow.com/questions/3454576/long-double-vs-double)  
* [Wikipedia](https://en.wikipedia.org/wiki/Long_double)  