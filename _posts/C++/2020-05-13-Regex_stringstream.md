---
title: C++ regex와 streamstring 사용법
date: 2020-05-13-11:00
categories:
- C++

tags:
- C++
- string
- 문자열
- 정규식
- regex
- streamstring

# photos: 
# - 주소

---

## C++에서 문자열을 좀더 편하게 다뤄봅니다.
> `regex`와 `stringstream`

---

## 글을 작성하기전에
> 이 글에서 다루는 내용은 아주 얕습니다.

* 저는 `C++`를 잘하지 못합니다.
* `PS`를 위한 사용법 정도만 간단히 다루어 봅니다.
* 나중에 자세히 알게되면 정리 포스팅을 추후 작성할 수...도 있습니다.

---

## Regex
> `#include<regex>`

* `C++ 11` 부터 정규식이 추가 되었습니다. 
* 원리 같은것도 파악하지 않고, 파싱을 위한 아주 기초적인 사용법만 익혀봅니다.

### 사용법

#### 정규식을 만들어줍니다. 
> `reg("정규식")`

```c++
regex reg("-?[0-9]+");
```

* 해당 정규식은 숫자를 파싱하는 정규식입니다.
* 정규식에 익숙치 않으시면 [RegExr](https://regexr.com/) 에서 연습하시기를 적극 권장합니다.

#### regex용 iterator 를 만듭니다.
> `stregex_iterator name;`

* 일치하는 정규식을 찾지못하는 경우 `end`를 반환합니다.
* 즉 찾지 못하는 경우를 처리하기 위해 `const sregex_iterator end;` 를 만들어 줍니다.
* 정규식을 찾아나갈 `start`를 만듭니다.
* sregex_iterator `start(string.begin(), string.end(), regex);`

```c++
const sregex_iterator end;
sregex_iterator start(input_str.begin(), input_str.end(), reg);
```

#### 매칭되는 문자열을 찾습니다.
> 본래는 `smatch`도 설명하고 사용해야 하나, 건너뜁니다.

```c++
while (start != end) {
	cout << start->str() << " ";
	start++;
}
cout << "\n";
```

---

### 예제 소스

```c++
#include<iostream>
#include<regex>
#include<sstream>
#include<string>

using namespace std;

void Get_Number_regex(string input_str){

	//regex를 이용한 정규식으로 숫자를 파싱하기
	cout << "\nregex를 이용한 정규식으로 숫자를 파싱하기\n";
	regex reg("-?[0-9]+");

	const sregex_iterator end;
	sregex_iterator start(input_str.begin(), input_str.end(), reg);
	
	while (start != end) {
		cout << start->str() << " ";
		start++;
	}
	cout << "\n";

	cout << "\nregex를 이용한 정규식으로 문자를 파싱하기\n";
	regex reg2("[A-z]+");
	sregex_iterator start2(input_str.begin(), input_str.end(), reg2);
	while (start2 != end) {
		cout << start2->str() << " ";
		start2++;
	}
	cout << "\n";
}

int main() {
	string str1 = "1A2B3CC4-5";
	
	cout << "input_str : " << str1 << "\n";
	//숫자를 추출해본다.
	Get_Number_regex(str1);

	return 0;
}
```

### 결과
```
input_str : 1A2B3CC4-5

regex를 이용한 정규식으로 숫자를 파싱하기
1 2 3 4 -5

regex를 이용한 정규식으로 문자를 파싱하기
A B CC
```

---

## stringstream
> `#include<sstream>`

* 타입과 매칭되는것이 나올때까지 탐색합니다.
* **중간에 매칭되는것이 없으면 종료됩니다.**
* 재사용을 위해서는 초기화 작업이 추가적으로 필요합니다.

### 사용법

#### 문자열을 받을 stringstream 을 만듭니다.
> `stringstream name(string)`

```c++
stringstream ss(input_str);
```

#### 찾을 타입을 정하고 순회합니다.
> `while (ss >> num)`

* `정수`, `실수`, `문자열` 등등 이 가능합니다.
* `cin` 후 `>>`이 console의 input을 stdin 에 넣는것처럼
* stringstream을 num에 넣어줍니다.

```c++
int num;
while (ss >> num) cout << num << " ";
cout << "\n";
```

#### 재사용을 위해서 초기화를 해줍니다.
> `stringstream_name.clear()`, `stringstream_name.str("")`

* `clear`를 해주어야 정상적으로 탐색합니다.

```c++
ss.clear();
ss.str(input_str);
```

---

### 예제 소스

```c++
#include<iostream>
#include<regex>
#include<sstream>
#include<string>

using namespace std;

void Get_Number_streamstring(string input_str) {
	stringstream ss(input_str);
	cout << "\nstringstream를 이용한 split 구현하기\n";


	cout << "정수형 : ";
	int num;
	while (ss >> num) cout << num << " ";
	cout << "\n";
	//정수이므로 소수점은 출력하지 못한다.
	
	ss.clear();
	ss.str(input_str);

	double dd;
	cout << "실수형 : ";
	while (ss >> dd) cout << dd << " ";
	cout << "\n";


	ss.clear();
	ss.str(input_str);

	string st;
	cout << "문자열 : ";
	while (ss >> st) cout << st << " ";
	cout << "\n";
}

int main() {

	string str2 = "1 .3 5 10 11";
	Get_Number_streamstring(str2);

	return 0;
}
```

---

### 결과

```
stringstream를 이용한 split 구현하기
정수형 : 1
실수형 : 1 0.3 5 10 11
문자열 : 1 .3 5 10 11
```

---

### 전체소스

```c++
#include<iostream>
#include<regex>
#include<sstream>
#include<string>

using namespace std;

void Get_Number_streamstring(string input_str) {
	stringstream ss(input_str);
	cout << "\nstringstream를 이용한 split 구현하기\n";


	cout << "정수형 : ";
	int num;
	while (ss >> num) cout << num << " ";
	cout << "\n";
	//정수이므로 소수점은 출력하지 못한다.
	
	ss.clear();
	ss.str(input_str);

	double dd;
	cout << "실수형 : ";
	while (ss >> dd) cout << dd << " ";
	cout << "\n";


	ss.clear();
	ss.str(input_str);

	string st;
	cout << "문자열 : ";
	while (ss >> st) cout << st << " ";
	cout << "\n";
}

void Get_Number_regex(string input_str){

	//regex를 이용한 정규식으로 숫자를 파싱하기
	cout << "\nregex를 이용한 정규식으로 숫자를 파싱하기\n";
	regex reg("-?[0-9]+");

	const sregex_iterator end;
	sregex_iterator start(input_str.begin(), input_str.end(), reg);
	
	while (start != end) {
		cout << start->str() << " ";
		start++;
	}
	cout << "\n";

	cout << "\nregex를 이용한 정규식으로 문자를 파싱하기\n";
	regex reg2("[A-z]+");
	sregex_iterator start2(input_str.begin(), input_str.end(), reg2);
	while (start2 != end) {
		cout << start2->str() << " ";
		start2++;
	}
	cout << "\n";
}

int main() {
	string str1 = "1A2B3CC4-5";
	
	cout << "input_str : " << str1 << "\n";
	//숫자를 추출해본다.
	Get_Number_regex(str1);
	cout << "\n====================\n";

	string str2 = "1 .3 5 10 11";
	Get_Number_streamstring(str2);


	return 0;
}
```

---

### 전체 결과

```
input_str : 1A2B3CC4-5

regex를 이용한 정규식으로 숫자를 파싱하기
1 2 3 4 -5

regex를 이용한 정규식으로 문자를 파싱하기
A B CC

====================

stringstream를 이용한 split 구현하기
정수형 : 1
실수형 : 1 0.3 5 10 11
문자열 : 1 .3 5 10 11
```

---

### 응용

```c++
#include<iostream>
#include<regex>
#include<sstream>
#include<string>

using namespace std;

void Get_Number_streamstring(string input_str) {
	stringstream ss(input_str);
	cout << "\nstramstring를 이용한 정규식으로 숫자문자를 파싱하기\n";

	int num;
	string str;
	while (ss >> num) {
		cout << num;
		ss >> str;
		cout << str;
		cout << " ";
	}
}

void Get_Number_regex(string input_str){

	cout << "\nregex를 이용한 정규식으로 숫자문자를 파싱하기\n";
	regex reg("-?[0-9][A-z]+");

	const sregex_iterator end;
	sregex_iterator start(input_str.begin(), input_str.end(), reg);
	
	while (start != end) {
		cout << start->str() << " ";
		start++;
	}
	cout << "\n";
}

int main() {
	string str1 = "1A 2BB 3CCC 4DDDD";
	
	cout << "input_str : " << str1 << "\n";


	Get_Number_regex(str1);
	cout << "\n====================\n";
	Get_Number_streamstring(str1);


	return 0;
}
```

### 응용 결과

```
input_str : 1A 2BB 3CCC 4DDDD

regex를 이용한 정규식으로 숫자문자를 파싱하기
1A 2BB 3CCC 4DDDD

====================

stramstring를 이용한 정규식으로 숫자문자를 파싱하기
1A 2BB 3CCC 4DDDD
```

---

## Reference

* https://word.tistory.com/24
* https://modoocode.com/