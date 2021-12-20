---
title: STL to Collection (Map, Set)
date: 2021-12-20-20:00
categories:
- Java

tags:
- Java
- Collection

---

## STL to Collection (1)
> cpp에서 사용하는 Map, Set을 java로 치환해봅니다.

---

# Map

## 기본 문법

> Cpp  


```cpp
unordered_map<int, bool> m;
map<int,bool> m; // key기준으로 정렬

m[key] = value; // input
m[key]; // output
if(used.find(3) != used.end()) // keyCheck
``` 

> java 


```java
Map<Integer, Boolean> m = new HashMap<>();
Map<Integer, Integer> m = new TreeMap<>(); // key기준으로 정렬

used.put(key, value); // input
used.get(key); // output
if(used.containsKey(key)); // keyCheck
```

## 순회하기

```cpp
int main() {
  ios_base::sync_with_stdio(false);
  cin.tie(0);

  unordered_map<int, string> used;
  used[1] = "apple";

  for (const auto&[k, v] : used) {
    cout << k << " " << v << "\n";
  }

  for (auto it : used) {
    cout << it.first << " " << it.second << "\n";
  }
}

```

```java
public static void main(String[] args) {
    Map<Integer, String> m = new HashMap<Integer, String>() {
        {
            put(1, "apple");
        }
    };

    for (Map.Entry<Integer, String> entry : m.entrySet()) {
        System.out.println(entry.getKey() + " " + entry.getValue());
    }

    for (Integer k : m.keySet()) {  // 이걸 제일 많이 쓸것 같다.
        System.out.println(k + " " + m.get(k));
    }

    m.forEach((k, v) -> System.out.println(k + " " + v));
}
```

---

## 없는 값에 접근하는 경우

```cpp
bool containsDuplicate(vector<int>& nums) {
    unordered_map<int, bool> used;
    for(int num : nums){
        if(used[num]) return true;
        used[num] = true;
    }
    return false;
}
```

```java
class Solution {
    public boolean containsDuplicate(int[] nums) {
        Map<Integer, Boolean> used = new HashMap<>();
        for (int num : nums) {
            if (used.containsKey(num)) {
                return true;
            }
            used.put(num, true);
        }
        return false;
    }
}
```

- cpp의 경우 `used[key]`로 없는 key값에 접근하는경우, 디폴트값이 자동으로 들어가게된다.
- 하지만 java의 경우 **Exception** 이 발생하게 된다.


---


# Set

## 기본 문법

> cpp  

```cpp
set<int> s;
s.insert(1);
cout << s.size() << "\n"; // 1

if (find(s.begin(), s.end(), 1) != s.end()) {
    cout << "exist" << "\n";    // exist
}

s.erase(1);
cout << s.size() << "\n"; // 0
```

> java


```java
Set<Integer> s = new HashSet<>();
s.add(1);
System.out.println(s.size());   // 1

if (s.contains(1)) {
    System.out.println("exist");    // exist
}

s.remove(1);
System.out.println(s.size());   // 0
```

## Array to Set

```cpp
bool containsDuplicate(vector<int>& nums) {
    set<int> uniqueNums {nums.begin(), nums.end()};
    return nums.size() != uniqueNums.size();
}
```

```java
public boolean containsDuplicate(int[] nums) {
    Set<Integer> uniqueNums = Arrays.stream(nums).boxed().collect(Collectors.toSet());
    return uniqueNums.size() != nums.length;
}
```

- Java의 경우 boxing 작업이 추가적으로 필요하다.
- Array(primitive) to List도 비슷한 방법으로 운영된다. 
- 이부분의 경우 자동완성을 사용하지 않는 상황에 대한 연습이 상당히 필요해보인다.

## MultiSet

```cpp
multiset<int> ms;

ms.insert(1);
ms.insert(1);
cout << ms.size() << "\n"; // 2;

ms.erase(1);  // 모든 1을 지움
cout << ms.size() << "\n"; // 0;

ms.insert(1);
ms.insert(1);
auto iter = ms.find(1);
ms.erase(iter); // 이터레이터에 해당하는 1개만 지움
cout << ms.size() << "\n"; // 1;
```


```java
// 없음. 
// 직접 구현해줘야함
Map<Integer, List<Integer>> ms = new TreeMap<>();
ms.put(1, Arrays.asList(1));
...
```

- java에서 `MultiSet`은 표준 Collection 으로 제공하지 않는다.
- `Guava` 같은 외부 라이브러리를 쓰는 수밖에 없다. (https://www.baeldung.com/guava-multiset)


---