---
title: STL to Collection (Permutation, Combination)
date: 2022-01-04-20:10
categories:
- Java

tags:
- Java
- Collection

---

## STL to Collection (3)
> cpp에서 사용하는 next_permutation 관련 기능을 java로 치환해봅니다.

---

## 순열(Permutation)


### cpp

```cpp
#include <bits/stdc++.h>

using namespace std;

int main() {
  vector<int> nums{1, 2, 3};

  do {
    for (int num : nums) {
      cout << num << " ";
    }
    cout << "\n";
  } while (next_permutation(nums.begin(), nums.end()));

  return 0;
}
```

```console
1 2 3 
1 3 2 
2 1 3 
2 3 1 
3 1 2 
3 2 1 
```

### java

```java
public class Permutation {

    public static void main(String[] args) {
        int[] arr = {1, 2, 3, 4};
        boolean[] used = new boolean[arr.length];
        int[] selected = new int[arr.length];

        permutation(arr, used, selected, 0);
    }

    private static void permutation(int[] arr, boolean[] used, int[] selected, int count) {
        if (count == arr.length) {
            printSelected(selected);
            return;
        }

        for (int index = 0; index < arr.length; ++index) {
            if (used[index]) {
                continue;
            }
            used[index] = true;
            selected[count] = arr[index];
            permutation(arr, used, selected, count + 1);
            used[index] = false;
        }
    }

    private static void printSelected(int[] nums) {
        for (int num : nums) {
            System.out.print(num + " ");
        }
        System.out.println();
    }
}
```

```console
1 2 3 
1 3 2 
2 1 3 
2 3 1 
3 1 2 
3 2 1 
```

- java의 경우 따로 메소드를 제공하지 않아서 직접 구현해주어야 합니다.

### kotlin

```kotlin
private fun printPermutation() {
    val list = listOf(1, 2, 3, 4)

    permutation(
        list = list,
        used = MutableList(list.size) { false },
        selected = MutableList(list.size) { -1 },
        count = 0
    )
}

private fun permutation(list: List<Int>, used: MutableList<Boolean>, selected: MutableList<Int>, count: Int) {
    if (count == list.size) {
        printSelected(selected)
        return
    }

    for (index in list.indices) {
        if (used[index]) continue

        used[index] = true
        selected[count] = list[index]
        permutation(list, used, selected, count + 1)
        used[index] = false
    }
}

private fun printSelected(nums: List<Int>) {
    for (num in nums) {
        print("$num ")
    }
    println()
}
```

- tip: `MutableList(list.size) { false }` 으로 원하는 사이즈만큼 디폴트 값을 채운채 초기화 할 수 있습니다.


---


## 조합(Combination)
> 조합의 경우 cpp, java 둘다 따로 메소드를 제공하지 않습니다.

### cpp

```cpp
#include <bits/stdc++.h>

using namespace std;

int main() {
  vector<int> nums{1, 2, 3, 4};
  vector<int> select{0, 0, 1, 1};
  //vector<int> select{1, 1, 0, 0}; (순서를 맞추고 싶은경우)

  do {
    for (int i = 0; i < (int) select.size(); ++i) {
      if (select[i] == 0) continue;
      cout << nums[i] << " ";
    }
    cout << "\n";
  } while (next_permutation(select.begin(), select.end()));
  //} while (prev_permutation(select.begin(), select.end()));   (순서를 맞추고 싶은경우)

  return 0;
}
```

- 저의 경우 cpp의 경우에는 `next_permutation` 를 이용해 작성합니다.
- 순서도 맞추고 싶은경우 `prev_permutation`을 이용하면됩니다.

```console
3 4 
2 4 
2 3 
1 4 
1 3 
1 2 
```

### java

```java
public class Combination {

    public static void main(String[] args) {
        int[] arr = {1, 2, 3, 4};
        boolean[] used = new boolean[arr.length];
        int wantCount = 2;

        combination(arr, used, wantCount, 0, 0);
    }

    private static void combination(int[] arr, boolean[] used, int wantCount, int selectedCount, int index) {
        if (selectedCount == wantCount) {
            printSelected(arr, used);
            return;
        }

        if (index == arr.length) {
            return;
        }

        used[index] = true;
        combination(arr, used, wantCount, selectedCount + 1, index + 1);

        used[index] = false;
        combination(arr, used, wantCount, selectedCount, index + 1);
    }

    private static void printSelected(int[] arr, boolean[] used) {
        for (int i = 0; i < used.length; ++i) {
            if (!used[i]) {
                continue;
            }
            System.out.print(arr[i] + " ");
        }
        System.out.println();
    }
}
```

```
1 2 
1 3 
1 4 
2 3 
2 4 
3 4 
```

- 선택하거나, 선택하지 않는 작업을 재귀적으로 반복합니다.
- 이때 항상 **인덱스는 증가**시킵니다.
- 기저 조건은 선택한 개수가 원하는 개수와 같을때 or 배열 인덱스를 넘어가려고 할때로 잡습니다. 두 순서를 주의해야합니다. 

### kotlin

```kotlin
private fun printCombination() {
    val list = listOf(1, 2, 3, 4)

    combination(
        list = list,
        used = MutableList(list.size) { false },
        wantCount = 2,
        selectedCount = 0,
        index = 0,
    )
}

private fun combination(list: List<Int>, used: MutableList<Boolean>, wantCount: Int, selectedCount: Int, index: Int) {
    if (selectedCount == wantCount) {
        printSelected(list, used)
        return
    }
    if (index == list.size) return

    used[index] = true
    combination(list, used, wantCount, selectedCount + 1, index + 1)
    used[index] = false
    combination(list, used, wantCount, selectedCount, index + 1)
}

private fun printSelected(nums: List<Int>, used: List<Boolean>) {

    nums.forEachIndexed { index, num ->
        if (used[index]) {
            print("$num ")
        }
    }
    println()
}
```