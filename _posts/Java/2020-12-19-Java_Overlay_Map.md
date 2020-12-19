---
title: JAVA 중첩된 map 사용시 주의할 점
date: 2020-12-19-23:00
categories:
- Java

tags:
- Java
- Collections


---

## Java로 중첩 map을 삽입시 주의할점을 기록해봅니다.
> map안에는 중복된 key가 들어갈 수 없다는것을 기억합니다.

---

## 중첩 map을 쓸때 주의하자.

* map안에 map을 넣는 경우를 생각해보자.
* 과거 C++ 로는 `[key1][key2] = value` 로 아주 간단히 처리했다.

```c++
#include <bits/stdc++.h>

using namespace std;

int main() {

	map<string, map<string, int>> m;
	m["김종완"]["정윤성"] = 28;
	m["김종완"]["김영호"] = 28;

	cout << m["김종완"]["정윤성"];	// 28
	return 0;
}
```

---

> 하지만 JAVA 로는 그렇지 못하다.

```java
package com.javapractice;

import java.util.HashMap;
import java.util.Map;

public class MapMapTest {

    public static Map<String, Integer> getMap(String name, int age) {
        Map<String, Integer> map = new HashMap<>();
        map.put(name, age);
        return map;
    }

    public static void main(String[] args) {
        Map<String, Map<String, Integer>> mmp = new HashMap<>();
        mmp.put("김종완", getMap("정윤성", 28));
        mmp.put("김종완", getMap("김영호", 28));    // 의도치 않은 작동
        
        System.out.println(mmp.get("김종완").get("정윤성"));  // null 이 나와버린다.
        // 생각해보면 김종완이라는 key를 가진 map은 하나밖에 될 수 없다.
        // 김영호 map을 넣을때 정윤성 map은 밀려서 없어져 버린다.

        mmp.clear();
        Map<String, Integer> innerMap = new HashMap<>();
        mmp.put("김종완", innerMap);

        innerMap.put("정윤성", 28);
        innerMap.put("김영호", 28);

        System.out.println(mmp.get("김종완").get("정윤성"));  // 28
    }
}
```

* `map1`안에 `map2`를 넣을때 잘 생각해보자.
* `map1`에서 사용할 수 있는 key는 하나 밖에 없다.
    * "김종완" 이 key로 쓰이는 map은 **단 한개**만 허용된다.
    * 여기서 "김종완" 을 또 다시 key로 쓰며 "김영호" 를 넣는것을 시도하면, 이전의 "정윤성"은 "김영호"에게 덮어 씌워지는 것이다.
    * 그래서 `System.out.println(mmp.get("김종완").get("정윤성"));` 는 `null` 이 나와버리는 것이다.

* 그렇다면 어떻게 해야할까.
    * `map1` 안에 **"김종완"** 을 key 값으로 하는 `map2` 를 넣어두고, `map2` 를 조작하면 된다!
    * 예제의 경우 `innerMap` 을 하나 만들어 놓고, 처리했다.
        * 이 경우, **"김종완"** 을  key 으로 쓰는 map은 하나만 되는것이고, 삽입된 map 에 새로운 key value를 넣는작업이 되므로 문제가 없다!

---

## 피드백

* 우테코 최종시험에서 이부분을 해결하지 못해서, 기능 구현을 하나 놓쳤다.
* 뿐만 아니라 이부분에 묶여 있다보니 다른 리팩토링 작업 같은것도 역시 하지 못했다.

* `될거 같은데 왜 안되지?` 라고 느낄때는, 
    * 일단 **침착** 하자.
    * 디버깅 모드를 켜보자.
    * 동작 방식을 한번 더 생각해보자.
        * 그래도 모르겠으면 `Ctrl + B` 로 내부를 좀 더 들여다 보자.