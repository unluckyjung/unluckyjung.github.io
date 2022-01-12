---
title: STL to Collection (BinarySearch, Gcd, Lcm)
date: 2022-01-12-16:06
categories:
- Java

tags:
- Java

---

## STL to Collection (7)
> cpp에서 사용하는 binarySearch 기능, gcd, lcm 을 java로 치환해 봅니다.

---

# BinarySearch

## cpp

```cpp
#include <bits/stdc++.h>

using namespace std;

int main() {
  vector<int> nums = {10, 20, 30, 40, 50, 60};
  cout << binary_search(nums.begin(), nums.end(), 10) << "\n";  // 1 (true);
  cout << binary_search(nums.begin(), nums.end(), 15) << "\n"; // 0 (false);

  // iterator로 반환하기 때문에 위치 인덱스를 얻고 싶다면, begin() 과의 차이를 구해야합니다.
  cout << upper_bound(nums.begin(), nums.end(), 30) - nums.begin() << "\n"; // 3
  cout << upper_bound(nums.begin(), nums.end(), 60) - nums.begin() << "\n"; // 6 (end 지점 none을 의미)
  cout << upper_bound(nums.begin(), nums.end(), 999) - nums.begin() << "\n"; // 6 (end 지점 none을 의미)

  cout << lower_bound(nums.begin(), nums.end(), 30) - nums.begin() << "\n"; // 2
  cout << lower_bound(nums.begin(), nums.end(), 10) - nums.begin() << "\n"; // 0
  cout << lower_bound(nums.begin(), nums.end(), 15) - nums.begin() << "\n"; // 1

  return 0;
}
```

## java
> `binarySearch`만 제공해줍니다. `lower, upper` 는 직접 구현해주어야 합니다.

```java
Collections.binarySearch(nums, targetNum);
// lowerBound, upperBound는 직접 구현
```

```java
import java.util.Arrays;
import java.util.Collections;
import java.util.List;

class Solution {

    private static List<Integer> nums = Arrays.asList(10, 20, 30, 40, 50, 60);

    public static void main(String[] args) {
        System.out.println(Collections.binarySearch(nums, 10)); // 0 (index)
        System.out.println(Collections.binarySearch(nums, 15)); // -2 (none)
        System.out.println(getUpperIndex(30));  // 3
        System.out.println(getUpperIndex(60));  // -1 (none)

        System.out.println(getLowerIndex(30));  // 2
        System.out.println(getLowerIndex(10));  // 0
        System.out.println(getLowerIndex(15));  // 1
    }

    private static int getUpperIndex(final int target) {
        int st = 0;
        int ed = nums.size() - 1;
        int index = -1;

        while (st <= ed) {
            int mid = st + (ed - st) / 2;
            if (nums.get(mid) > target) {
                index = mid;
                ed = mid - 1;
            } else {
                st = mid + 1;
            }
        }
        return index;
    }

    private static int getLowerIndex(final int target) {
        int st = 0;
        int ed = nums.size() - 1;
        int index = -1;

        while (st <= ed) {
            int mid = st + (ed - st) / 2;
            if (nums.get(mid) >= target) {
                index = mid;
                ed = mid - 1;
            } else {
                st = mid + 1;
            }
        }
        return index;
    }
}
```

---

# Gcd, Lcm

## cpp
> `cpp 17` 이상부터 지원합니다.

```cpp
#include <bits/stdc++.h>

using namespace std;

int main() {
  int num1 = 8;
  int num2 = 15;

  cout << gcd(8, 12) << "\n";   // 4
  cout << lcm(8, 12) << "\n";   // 24

  return 0;
}
```

## java
> `java 11` 기준으로 제공하지 않습니다. 직접 구현 해주어야합니다. [유클리드 호제법](https://ko.wikipedia.org/wiki/%EC%9C%A0%ED%81%B4%EB%A6%AC%EB%93%9C_%ED%98%B8%EC%A0%9C%EB%B2%95)으로 직접 구할 수 있습니다.

```java
public class Solution {

    public static void main(String[] args) {
        int num1 = 8;
        int num2 = 12;
        System.out.println(gcd(num1, num2));    // 4
        System.out.println(lcm(num1, num2));    // 24
    }

    private static int gcd(final int num1, final int num2) {
        if (num2 == 0) return num1;
        return gcd(num2, num1 % num2);
    }

    private static int lcm(final int num1, final int num2) {
        return num1 * num2 / gcd(num1, num2);
    }
}
```