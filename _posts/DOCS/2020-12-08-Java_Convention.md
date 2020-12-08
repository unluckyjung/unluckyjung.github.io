---
title: 구글 자바 컨벤션 정리
date: 2020-01-01-00:00
categories:
- DOCS

tags:
- Java
- Convention

photos: 
- 주소

---

## 구글의 자바 컨벤션을 정리해봅니다.
> 전부 말고, 제가 자주 실수하는 부분만 기록합니다.

---

## 구글 자바 컨벤션 주소
> [원글 주소](https://google.github.io/styleguide/javaguide.html)

* 한번 정리해야지.. 정리해야지 치일피일 미루다가, 확실하게 기억할겸 문서화 합니다.
    * 계속 업데이트 될 수 있습니다.
    * 오역이 있을 수 있습니다.

---

## 포매팅

### 괄호
* 괄호는 선택사항에서도 쓰입니다.
    * `if`, `else`, `for`, `do`, `while` 에 전부 쓰입니다.
    * 단 한줄이라도 괄호를 사용됩니다.
        * `if (true) return "Yes"` 가 허용되는 `C++ Style` 과 다릅니다.
    * 몸체가 없는 비어있는 경우에도 괄호가 쓰입니다.

```java
return () -> {
  while (condition()) {
    method();
  }
};

return new MyClass() {
  @Override public void method() {
    if (condition()) {
      try {
        something();
      } catch (ProblemException e) {
        recover();
      }
    } else if (otherCondition()) {
      somethingElse();
    } else {
      lastThing();
    }
  }
};
```
* `else`, `if else`, `catcth` 같은경우는 줄바꿈을 하지 않습니다.


```java
  // This is acceptable
  void doNothing() {}

  // This is equally acceptable
  void doNothingElse() {
  }
```

* 몸체가 빈 경우에는 간결하게 한줄로 사용할 수 있습니다.

```java
  // This is not acceptable: No concise empty blocks in a multi-block statement
  try {
    doSomething();
  } catch (Exception e) {}
```

* 하지만 다음과 같이 멀티 블록 구문에서는 한줄로 사용할 수 **없습니다.**

### 공백
* `수직 공백`은 어디에든 쓰일 수 있습니다.
    * 논리적 부분으로 나눌때 쓰일 수 있습니다.
    * 하지만 다수의 공백줄은 사용할 순 있으나 `지양` 합니다.

### 변수 선언
* 정의당 하나의 변수만 사용합니다.

```java
// 허용되지 않습니다.
int num1, num2 

// 다음과 같이 선언합니다
int num1;
int num2;
```

* 하지만 for 루프에서는 예외로 허용됩니다.

---

## 네이밍


### 패키지
* 패키지 이름은 다음과 같이한다.
    * 전부 소문자로 이루어진다.
    * 언더스코어를 허용하지 않는다.

```java
// ok
pakage com.exampledeepspace

// no
pakage com.example.deepSpace
pakage com.example.depp_sapce
```

### 클래스
* `UpperCamelCase` 로 작성한다.
* 클래스 이름은 명사나, 명사 구로 작성한다.
    * 인터페이스도 마찬가지이다.
    * 종종, 형용사구가 사용될 수 도 있다.
    * 테스트 용의 경우에는, 끝에 Test를 붙여준다.

```java
class Reader;
class Readable;
class ReaderTest;
```

### 메소드
* `lowerCamelCase` 로 작성한다.
* 메소드 이름은 동사, 동사 구로 작성된다.

## 상수
* `CONST_CASE` 로 작성한다.
* 전형적으로 명사로 작성한다.