---
title: STL to Collection (String)
date: 2022-01-04-20:15
categories:
- Java
- Kotlin

tags:
- Java
- Kotlin
- Collection

---

## STL to Collection (4)
> cpp에서 사용하는 string 관련기능을 java, kotlin 로 치환해 봅니다.

### Java, Kotlin 문자열 관련 객체들에 대해 알아둘점

> java

```java
String str = "JungYoonsung"; // immutable + user String constant pool 
// str == "JungYoonsung" (true)

String str1 = new String("JungYoonsung"); // immutable
// str1 == "JungYoonsung" (false)

StringBuilder stringBuilder = new StringBuilder(str); // mutable + thread unsafe
StringBuffer stringBuffer = new StringBuffer(str); // mutable + thread safe
// 수행 속도는 StringBuiler쪽이 뛰어나다. PS 단에서는 StringBuilder를 사용할것
```

> kotlin

```kotlin
val str = "JungYoonsung"
val stringBuilder = StringBuilder("JungYoonsung")
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

### kotlin

```kotlin
var str = "JungYoonsung"

println(str.substring(4)) // Yoonsung
println(str.substring(4, 5)) // Y (두번째 인자는 닫힌 index 이다.)
println(str.indexOf("Yoon")) // 4

if (str.contains("null").not()) {
    println("NOT FOUND") // "NOT FOUND"
}

str += "UnlcukyJung"
println(str) // "JungYoonsungUnlcukyJung"
```



## 문자열 대소 비교시 (java, kotlin)

```java
String str = "aaa";
System.out.println(str.compareTo("bbb"));   // -1
System.out.println(str.compareTo("bb"));   // -1
System.out.println(str.compareTo("aaaa"));   // -1

System.out.println(str.compareTo("aaa"));   // 0

System.out.println(str.compareTo("aa"));   // 1 (1이상의 값)
System.out.println(str.compareTo("a"));   // 2 (1이상의 값)
```

```kotlin
val str = "aaa"

println(str.compareTo("bbb")) // -1
println(str < "bbb")  // true

println(str.compareTo("bb")) // -1
println(str.compareTo("aaaa")) // -1

println(str.compareTo("aaa")) // 0

println(str.compareTo("aa")) // 1 (1이상의 값)
println(str > "aa")  // true

println(str.compareTo("a")) // 2 (1이상의 값)
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

### kotlin

```kotlin
val str = "JungYoonsung"
val strBuilder = StringBuilder("JungYoonsung")

println(str[3]) // g
println(strBuilder[3]) // g
```

- kotlin 의 경우에는 cpp 와 마찬가지로 `[]` 로 접근할 수 있습니다.

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

### kotlin

```kotlin
val str = "JungYoonsung"
val strBuilder = StringBuilder(str)

for (ch in str.toCharArray()) {
    println(ch)
}
println()

for (ch in str) {
    println(ch)
}
println()

for (i in 0 until str.length) {
    println(str[i])
    println(strBuilder[i])
}
println()

for (i in str.indices) {
    println(str[i])
    println(strBuilder[i])
}
println()

str.forEachIndexed { index, ch ->
    println("index: $index, value: $ch")
}
```

- kotlin 의 경우에는 `forEachIndexed` 와 `indices` 를 이용하여 인덱스 기반의 조회를 쉽게 할 수 있습니다.

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

### kotlin

```kotlin
val phoneNumber = "123-456-789"
val split = phoneNumber.split("-")
split.forEach {
    println(it) // {123, 456, 789}
}
```

---

## 문자열 뒤집기

### cpp

```cpp
string str = "abcd";
reverse(str.begin(), str.end());
cout << str;  // "cdba"
```

### java

```java
StringBuilder sb = new StringBuilder("abcd");
sb.reverse();
System.out.println(sb); // "dcba"
```

### kotlin

```kotlin
val sb = StringBuilder("abcd")
sb.reverse()
println(sb) // "dcba"
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

### kotlin

```kotlin
val strNumber = "0123"
val intNumber = strNumber.toInt()
println(intNumber) // (int) 123

println(intNumber.toString()) // (string) "123"
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
```

### kotlin

```kotlin
// String
val str1 = "abcdef"

println(str1.substring(0, 4)) // abcd

println(str1.replace("ef", "")) // abcd
println(str1.removeRange(2, 4)) // ab(cd)ef     // abef    // 두번쨰 인자는 inclusive

// StringBuilder
var strBuilder = StringBuilder("abcdef")
println(strBuilder.deleteCharAt(4)) // index 4 만 삭제  abcd(e)f    // abcdf

strBuilder = StringBuilder("abcdef")
println(strBuilder.delete(4, 6)) // abcd(ef)    // abcd

strBuilder = StringBuilder("abcdef")
println(strBuilder.delete(4, 100)) // abcd(ef)    // abcd

strBuilder = StringBuilder("abcdef")
println(strBuilder.deleteAt(4)) // index 4만 삭제  abcd(e)f    // abcdf

val str2 = StringBuilder("cd")
println(str2.replace(Regex("d"), "zzz"))    // czzz
```

- kotlin 의 String 에서는 `removeRange` 를 지원합니다.
- StringBuilder replace 사용시, target String 은 Regex 로 주어야 합니다.

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

```kotlin
val str = "Abcd"

if (str[0].isLetter()) {
    println("alpha") // alpha
}

if (str[0].isUpperCase()) {
    println("upperCase") // upperCase
}

println(str[0].lowercase()) // A-> a    // a
println(str[0].lowercaseChar()) // A-> a    // a

println(str.uppercase()) // "ABCD"
println(str.lowercase()) // "abcd

val numStr = "0A"
println(numStr[0].isDigit())  // true
numStr.forEach {
    println(it.isLetterOrDigit())
}
```

- `kotlin` 의 경우 `Character` 를 사용하지 않아도 됩니다.

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

### kotlin

```kotlin
val str = "JungYoonSung"
println(str.matches(Regex("JungYoonSung"))) // true
println(str.matches(Regex("unluckyJung"))) // false

val numberRegex = Regex("\\d+")

val str1 = "A1B2"
println(numberRegex.containsMatchIn(str1))  // true (매칭되는 1, 2이 있음)
println(numberRegex.matches(str1))  // false

val matchNumber = numberRegex.find(str1)
println(matchNumber?.value) // 앞의 1만 추출 // 1

val matchNumbers = numberRegex.findAll(str1)
matchNumbers.forEach {
    println(it.value)   // [1,2]
}
```

- kotlin 의 경우 java 보다 훨씬 편하게 정규식을 다룰 수 있습니다.
- `containsMatchIn` 을 통해 정규식에 맞는 케이스가 하나라도 있는지 확인할 수 있습니다.
- `matches` 를 통해 완벽히 매칭되는 케이스가 있는지 확인할 수 있습니다.
- `find` 를 통해서 가장 빨리 매칭되는 결과를 찾을 수 있습니다.
- `findAll` 를 통해서 매칭되는 모든 결과를 찾을 수 있습니다.

> kotlin 정규식을 다루는 방법은 다른 포스팅으로 작성할 예정입니다. `23.02.12`

---

## Reference
- [String](https://docs.oracle.com/javase/8/docs/api/index.html?java/util/regex/package-summary.html)
- [StringBuilder](https://docs.oracle.com/javase/8/docs/api/index.html?java/util/regex/package-summary.html)
- [StringBuffer](https://docs.oracle.com/javase/8/docs/api/index.html?java/util/regex/package-summary.html)
- [matches](https://docs.oracle.com/javase/8/docs/api/index.html?java/util/regex/package-summary.html)
- [kotlin regex](https://codechacha.com/ko/kotlin-how-to-use-regex/)