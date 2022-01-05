---
title: STL to Collection (String)
date: 2022-01-04-20:15
categories:
- Java

tags:
- Java
- Collection

---

## STL to Collection (4)
> cpp에서 사용하는 string 관련기능을 java로 치환해 봅니다.

### Java 문자열 관련 객체들에 대해 알아둘점

```java
String str = "JungYoonsung"; // immutable + user String constant pool 
// str == "JungYoonsung" (true)

String str1 = new String("JungYoonsung"); // immutable
// str1 == "JungYoonsung" (false)

StringBuilder stringBuilder = new StringBuilder(str); // mutable + thread unsafe
StringBuffer stringBuffer = new StringBuffer(str); // mutable + thread safe
// 수행 속도는 StringBuiler쪽이 뛰어나다. PS 단에서는 StringBuilder를 사용할것
```

- 불변, String constatnt pool, 스레드 세이프 한 부분들을 고려해야 합니다.

---

## 기본 문법
> substring, find, indexOf, append

### cpp

```cpp
string str = "JungYoonsung";
cout << str.substr(4) << "\n"; //Yoonsung
cout << str.substr(4, 1) << "\n"; //Y

cout << str.find("Yoon") << "\n"; // 4  Jung(Y) 4번째 index
if (str.find("null") == -1) {
  cout << "NOT FOUND" << "\n";  // "NOT FOUND"
}
if (str.find("null") == string::npos) {
  cout << "NOT FOUND" << "\n";  // "NOT FOUND"
}

str += "UnlcukyJung"; // "JungYoonsungUnlcukyJung"
```

### java

```java
String str = "JungYoonsung";
System.out.println(str.substring(4)); // Yoonsung
System.out.println(str.substring(4, 5));  // 두번째 인자는 닫힌 index이다.

System.out.println(str.indexOf("Yoon"));  // 4
if (!str.contains("null")) {
    System.out.println("NOT FOUND");  // "NOT FOUND"
}

str += "UnlcukyJung"; // "JungYoonsungUnlcukyJung"
```

## 문자열 대소 비교시 (java)

```java
String str = "aaa";
System.out.println(str.compareTo("bbb"));   // -1
System.out.println(str.compareTo("bb"));   // -1
System.out.println(str.compareTo("aaaa"));   // -1

System.out.println(str.compareTo("aaa"));   // 0

System.out.println(str.compareTo("aa"));   // 1 (1이상의 값)
System.out.println(str.compareTo("a"));   // 2 (1이상의 값)
```

---

## N번째 index에 접근하기

### cpp
```cpp
string str = "JungYoonsung";
cout << str[3]; //g   Jun(g)
```

### java

```java
String str = "JungYoonsung";
StringBuilder strBuilder = new StringBuilder("JungYoonsung");

System.out.println(str.charAt(3));  // g
System.out.println(strBuilder.charAt(3)); // g
```

---

## 문자열 -> 문자 순회

### cpp

```cpp
string str = "JungYoonsung";

for (char ch : str) {
cout << ch << "\n";
}

for (int i = 0; i < (int)str.size(); ++i) {
    cout << str[i] << "\n";
}
```

### java

```java
String str = "JungYoonsung";
StringBuilder strBuilder = new StringBuilder(str);

for (char ch : str.toCharArray()) {
    System.out.println(ch);
}

for (int i = 0; i < str.length(); ++i) {
    System.out.println(str.charAt(i));
    System.out.println(strBuilder.charAt(i));
}
```

---

## split

### cpp

```cpp
vector<string> split(const string &str, char delimiter) {
  vector<string> vec;
  stringstream ss(str);

  string splitStr;
  while (getline(ss, splitStr, delimiter)) {
    vec.push_back(splitStr);
  }
  return vec;
}

int main() {
  string phoneNumber = "123-456-789";
  char delimiter = '-';
  vector<string> nums = split(phoneNumber, delimiter);  // {123, 456, 789}

  return 0;
}
```

### java

```java
public class Main {

    public static void main(String[] args) {
        String phoneNumber = "123-456-789";
        String[] split = phoneNumber.split("-");  // {123, 456, 789}
    }
}
```


---

## 숫자관련 형변환

### cpp

```cpp
string strNumber = "0123";
int intNumber = stoi(strNumber);
cout << intNumber << "\n"; // (int) 123
cout << to_string(intNumber) << "\n"; // (string) "123"
```

### java

```java
String strNumber = "0123";
int intNumber = Integer.parseInt(strNumber);
System.out.println(intNumber);  // (int) 123
System.out.println(Integer.toString(intNumber));  // (string) "123"
```
---

## 변환, 치환, 삭제

### cpp

```cpp
string str = "abcdef";

str.erase(4); // abcd(ef)
cout << str << "\n"; // abcd

str.erase(0, 2); // (ab)cd
cout << str << "\n"; // cd

str.replace(1, 1, "zzz"); // c(d)
cout << str << "\n"; // czzz

str.insert(1, "AAA"); // c(z)zz
cout << str << "\n"; // cAAAzz
```

### java

```java
String str = "abcdef";
StringBuilder strBuilder = new StringBuilder("abcdef");

// String은 따로 삭제 메서드를 제공하지 않는다.
// https://www.baeldung.com/java-remove-replace-string-part
System.out.println(str.substring(0, 4));    // abcd
System.out.println(str.replace("ef", ""));  // abcd
System.out.println(strBuilder.deleteCharAt(4)); // index 4만 삭제  abcd(e)f    // abcdf

String str = "cd";
System.out.println(str.replace("d", "zzz"));  // czzz
```

---

## upper, alpha check

### cpp

```cpp
string str = "Abcd";

if (isalpha(str[0])) {
  cout << "alpha" << "\n";  // alpha
}
if (isupper(str[0])) {
  cout << "upperCase" << "\n";  // upperCase
}
cout << char(tolower(str[0])) << "\n";  // a
```

### java

```java
String str = "Abcd";
if (Character.isAlphabetic(str.charAt(0))) {
    System.out.println("alpha");    // alpha
}
if(Character.isUpperCase(str.charAt(0))){
    System.out.println("upperCase");    // upperCase
}
System.out.println(Character.toLowerCase(str.charAt(0)));   // a
System.out.println(str.toUpperCase());  // "ABCD"
System.out.println(str.toLowerCase());  // "abcd"
```

---

## 정규식 관련
> 문자열의 정규식 매칭 유무를 따져봅니다.

### cpp

```cpp
regex reg("JungYoonSung");

cout << regex_match("JungYoonSung", reg) << "\n"; // 1 (true)
cout << regex_match("unluckyJung", reg) << "\n";  // 0 (false)
```

### java

```java
String str = "JungYoonSung";
System.out.println(str.matches("JungYoonSung"));    // true
System.out.println(str.matches("unluckyJung"));     // false


Pattern pattern = Pattern.compile("JungYoonSung");
System.out.println(pattern.matcher("JungYoonSung").matches());  // true
System.out.println(pattern.matcher("unluckyjung").matches());   // false
```

---

## Reference
- [String](https://docs.oracle.com/javase/8/docs/api/index.html?java/util/regex/package-summary.html)
- [StringBuilder](https://docs.oracle.com/javase/8/docs/api/index.html?java/util/regex/package-summary.html)
- [StringBuffer](https://docs.oracle.com/javase/8/docs/api/index.html?java/util/regex/package-summary.html)
- [matches](https://docs.oracle.com/javase/8/docs/api/index.html?java/util/regex/package-summary.html)