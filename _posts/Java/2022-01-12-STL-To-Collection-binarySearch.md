---
title: STL to Collection (BinarySearch, Gcd, Lcm)
date: 2022-01-12-16:06
categories:
- Java
- Kotlin

tags:
- Java
- Kotlin

---

## STL to Collection (7)
> cpp에서 사용하는 binarySearch 기능, gcd, lcm 을 java, kotlin 로 치환해 봅니다.

---

## BinarySearch

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

## Kotlin

### binary Search

```kotlin
private fun binarySearch() {
    val nums = listOf(10, 20, 30, 40, 50, 60)
    println(nums.binarySearch(30))  // 2
    println(nums.binarySearch(-2))  // -1

    println(nums.binarySearch(element = 40, fromIndex = 0, toIndex = 4))    // 3
    println(nums.binarySearch(element = 40, fromIndex = 0, toIndex = 3))    // 음수 값
    println(nums.binarySearch(element = 40, fromIndex = 3, toIndex = 3))    // 음수 값
}
```

- kotlin 의 경우도 `binarySearch` 는 기본적으로 제공해 주고 있습니다.
- index 를 기반으로 range 를 정해줄 수 있으나 toIndex 는 Exclusive 합니다.

![image](https://user-images.githubusercontent.com/43930419/219856069-db2ed9e0-f647-46a9-a1a3-d9702cfcb4ab.png)

- 구현체를 보면 전형적인 binarySearch 를 구현해주고 있는것을 볼 수 있습니다. (`ushr` 은 코틀린에서 제공하는 비트 연산 함수 입니다.)

### lower, upper bound

```kotlin

fun lowerUpperBound() {
    val nums = listOf(10, 20, 30, 40, 50, 60)

    println(getLowerIndex(nums = nums, -2)) // 0
    println(getLowerIndex(nums = nums, 30)) // 2
    println(getLowerIndex(nums = nums, 60)) // 5

    println(getUpperIndex(nums = nums, -2)) // 0
    println(getUpperIndex(nums = nums, 30)) // 3
    println(getUpperIndex(nums = nums, 60)) // -1
}

private fun getUpperIndex(nums: List<Int>, target: Int): Int {
    var st = 0
    var ed: Int = nums.size - 1
    var index = -1

    while (st <= ed) {
        val mid = st + (ed - st) / 2
        if (nums[mid] > target) {
            index = mid
            ed = mid - 1
        } else {
            st = mid + 1
        }
    }
    return index
}

private fun getLowerIndex(nums: List<Int>, target: Int): Int {
    var st = 0
    var ed: Int = nums.size - 1
    var index = -1

    while (st <= ed) {
        val mid = st + (ed - st) / 2
        if (nums[mid] >= target) {
            index = mid
            ed = mid - 1
        } else {
            st = mid + 1
        }
    }
    return index
}
```

- kotlin 의 경우에도 lower, upper bound 는 직접 구현해주어야 합니다.

---

## Gcd, Lcm

### cpp
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

### java
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

### kotlin

```kotlin
fun main() {
    println(gcd(8, 12)) // 4
    println(lcm(8, 12)) // 24
}

private fun gcd(num1: Int, num2: Int): Int {
    return if (num2 == 0) num1 else gcd(num2, num1 % num2)
}

private fun lcm(num1: Int, num2: Int): Int {
    return num1 * num2 / gcd(num1, num2)
}
```

- kotlin 의 경우에도 직접 구현해주어야 합니다.