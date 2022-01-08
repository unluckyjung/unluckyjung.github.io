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

### cpp  
```cpp
unordered_map<int, bool> m;
map<int,bool> m; // key기준으로 정렬

m[key] = value; // input
m[key]; // output
if(used.find(3) != used.end()) // keyCheck
``` 

### java 


```java
Map<Integer, Boolean> m = new HashMap<>();
Map<Integer, Integer> m = new TreeMap<>(); // key기준으로 정렬

used.put(key, value); // input
used.get(key); // output
if(used.containsKey(key)); // keyCheck
```

## 순회하기

### cpp

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

### java

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

    for (Integer k : m.keySet()) {  
        System.out.println(k + " " + m.get(k));
    }

    m.forEach((k, v) -> System.out.println(k + " " + v));
}
```

- 개인적으로 `keySet()` 을 쓰는것이 제일 편하다고 느낍니다.
- 참고로 java에서 Map은 Collection으로 취급하지 않습니다. 

```
Map Interface
Why doesn't Map extend Collection?
This was by design. We feel that mappings are not collections and collections are not mappings. Thus, it makes little sense for Map to extend the Collection interface (or vice versa).

If a Map is a Collection, what are the elements? The only reasonable answer is "Key-value pairs", but this provides a very limited (and not particularly useful) Map abstraction. You can't ask what value a given key maps to, nor can you delete the entry for a given key without knowing what value it maps to.

Collection could be made to extend Map, but this raises the question: what are the keys? There's no really satisfactory answer, and forcing one leads to an unnatural interface.

Maps can be viewed as Collections (of keys, values, or pairs), and this fact is reflected in the three "Collection view operations" on Maps (keySet, entrySet, and values). While it is, in principle, possible to view a List as a Map mapping indices to elements, this has the nasty property that deleting an element from the List changes the Key associated with every element before the deleted element. That's why we don't have a map view operation on Lists.
```

- [공식문서 링크](https://docs.oracle.com/javase/8/docs/technotes/guides/collections/designfaq.html#a14)
- Map의 경우 Key와 value를 쌍으로 저장하기 떄문에, Collection 인터페이스의 의미가 모호해진다는 문제가 있기 때문입니다.

---

## 없는 값에 접근하는 경우

### cpp
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

### java
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

- cpp의 경우 `used[key]`로 없는 key값에 접근하는경우, 디폴트값이 자동으로 들어가게 됩니다.
- 하지만 java의 경우 **Exception** 이 발생하게 됩니다.

> `getOrDefefault(key, defaultValue)`

```java
import java.util.Arrays;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

class Solution {
    public static void main(String[] args) {
        List<Integer> nums = Arrays.asList(1, 2, 1, 1);
        Map<Integer, Integer> usedCount = new HashMap<>();
        for (int num : nums) {
            usedCount.put(num, usedCount.getOrDefault(num, 0) + 1);
        }
        System.out.println(usedCount.get(1));   // 3
    }
}
```

- `getOrDefault()` 를 이용해 공수를 좀 더 줄일 수 있습니다.


---

# Set

## 기본 문법

### cpp  

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

### java


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

### cpp
```cpp
bool containsDuplicate(vector<int>& nums) {
    set<int> uniqueNums {nums.begin(), nums.end()};
    return nums.size() != uniqueNums.size();
}
```

### java

```java
public boolean containsDuplicate(int[] nums) {
    Set<Integer> uniqueNums = Arrays.stream(nums).boxed().collect(Collectors.toSet());
    return uniqueNums.size() != nums.length;
}
```

- 일반적으로 많은 **저지 사이트**에서 cpp의 경우 `vector`를 인자로 제공하지만, Java의 경우 `int[]`를 인자로 제공합니다.
- 즉 Java의 경우 boxing 작업이 추가적으로 필요해지는 경우가 종종 발생합니다.
- Array(primitive) to List도 비슷한 방법으로 운영됩니다.

```java
List<Integer> list = Arrays.asList(1,2,3);
Set<Integer> set = new HashSet<>(list);
```

- List인 경우 심플하게 해결이 되는데, 좀 아쉬운 부분입니다.

## MultiSet

### cpp
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

### java
```java 
// 없음 직접 구현해줘야함
Map<Integer, List<Integer>> ms = new TreeMap<>();
ms.put(1, Arrays.asList(1));
...
```

- java에서 `MultiSet`은 표준 Collection 으로 제공하지 않습니다.
- `Guava` 같은 [외부 라이브러리를 쓰는 수밖에 없습니다.](https://www.baeldung.com/guava-multiset)


---