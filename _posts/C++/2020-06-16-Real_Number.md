---
title: C++ 정수, 실수 확인
categories:
- C++

tags:
- C++
- double
- float
- real-number

---

## 실수가 정수인지 소수인지 확인해봅니다.
> `1.0` 은 정수로 표현 될 수 있고, `1.1`은 그렇지 못합니다.

---

## 먼저 실수의 비교 연산을 해봅니다.
> 실수끼리 `==` 를 사용하는 방법을 알아 봅니다.

* 컴퓨터에서 실수는 오차없이 정확히 나타낼 수 없습니다.
* 즉, 실수끼리 완벽하게 같은지 확인할 수 없습니다.
* 하지만, 부동 소수점 오차 범위내에서는 확인 할 수 있습니다.

> :heavy_check_mark: 두개의 수를 뺀후, 절대값의 차이가 일정 이하인 경우 같다고 표현해 봅니다.

```c++
#include<iostream>
using namespace std;


int main() {

	if (0.1 + 0.1 + 0.1 == 0.3) cout << "yes\n";
	else cout << "no\n";

	if (abs((0.1 + 0.1 + 0.1) - 0.3) < 1e-12)cout << "yes\n";
	else cout << "no\n";

	return 0;
}
```

```
no
yes
```

---

## 위에서 이용한 방법을 응용해 봅니다.
> :heavy_check_mark: 기존의 수와, 소수점을 뗀 수를 뺀 후 비교 해봅니다.

```c++
#include<iostream>
using namespace std;


int main() {
	double d;

	d = 1.00;
	if (abs(d - (int)d < 1e-12)) cout << "Int\n";
	else cout << "double\n";

	d = 1.000001;
	if (abs(d - (int)d < 1e-12)) cout << "int\n";
	else cout << "double\n";

	return 0;
}
```

```
Int
double
```

---

## 부록

### 출력되는 소수점의 자릿수를 제한 해봅니다.


### precision
> :heavy_check_mark: `cout.precision(num)`

* `default`시 출력 되는 숫자의 `전체` 자리수를 제한합니다.

### fixed
> :heavy_check_mark: `cout << fixed`
* 출력되는 소수점 아래를 조절합니다.


```c++
#include<iostream>
using namespace std;


int main() {
	double d = 123.4567890;
	cout << d << "\n";

	cout.precision(4);
	cout << d << "\n";

	cout << fixed;
	cout.precision(2);
	cout << d << "\n";

	return 0;
}
```

```
123.457
123.5
123.46
```