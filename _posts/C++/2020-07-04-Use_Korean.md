---
title: C++ 에서 한글을 처리하기
date: 2020-07-04-19:00
categories:
- C++

tags:
- C++

---

## C++ 에서 한글을 처리해봅니다.
> 굉장히 불편하지만 가능합니다.

---

## 일반적인 생각
> `char` 타입에 한글자씩 넣자!

```c++
#include <iostream>
#include <string>

using namespace std;

int main()
{
    string korean = "정윤성";
    cout << korean[0] << "\n";
	return 0;
}
```

* `korean[0]` 의 리턴값으로 `"정"` 을 기대해볼만 합니다.

### 결과

```console
?
```
---

* 전혀 이상한 값이 출력됩니다.
* 생각을 해보면, 한글은 [ASCII 코드표](https://ko.wikipedia.org/wiki/ASCII)에 없는것을 기억해낼 수 있습니다.

---

## 길이를 재보자
> 한글이 들어가 있는 변수의 크기를 출력해봅니다.

```c++
#include <iostream>
#include <string>

using namespace std;

int main()
{
    string korean = "정윤성";
    cout << (int)korean.size() << "\n";
	return 0;
}
```

### 결과

```console
9
```

---
* `C++14 (gcc 8.3)` 기준으로, `9` 라고 찍힙니다.
	* :heavy_exclamation_mark: `visual studio` 에서는 `2byte` 로 결정됩니다.
* 이를통해 한글의 한글자 크기는 `3byte`라는것을 알수 있게됩니다.

---

## 이제 한글을 출력해보자
> `3개`를 연달아 출력

```c++
int main()
{
    string korean = "정윤성";
    cout << korean[0] << korean[1] << korean[2] << "\n";
	return 0;
}
```

### 결과

```console
정
```

---

## 한글자씩 전체를 출력해보자.
> `substring` 을 이용한 `3개`씩 끊어서 출력

```c++
#include <iostream>
#include <string>

using namespace std;

int main()
{
    string korean = "정윤성";
    for(int i = 0; i < (int)korean.size(); i += 3){
        cout << korean.substr(i, 3);
    }
    
    return 0;
}
```

### 결과

```console
정윤성
```

---

## 한글을 벡터에 저장해보자
> 잘라낸것을 `string` 타입으로 하나씩 저장.

```c++
#include <iostream>
#include <string>
#include <vector>
 
using namespace std;
 
int main()
{
    string korean = "정윤성";
 
    vector<string> koreanVec;
 
 
    for(int i = 0; i < (int)korean.size(); i += 3){
        koreanVec.push_back(korean.substr(i, 3));
    }
 
    for(const string &ko : koreanVec){
        cout << ko << " ";
    }
 
    return 0;
}
```

### 결과

```console
정 윤 성
```

---