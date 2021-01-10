---
title: [JAVA] String equals시 주의할점
date: 2021-01-10-22:00
categories:
- Java

tags:
- Java
- String


---

## Java에서 String equals 시 주의할점을 적어봅니다.
> Null 참조를 주의합니다

---

## 일반적으로 사용하는방법, 변수를 먼저 쓴다.
> `변수.equals("상수")`

* 해당 방법은 변수가 **null**이 들어올 경우 예외를 발생시키게 되고, 비정상적인 프로그램 종료를 발생시킬 수 있다.

```java
    public static void equalsTest1(String name) {
        if (name.equals("JYS")) {   // null exception!
            System.out.println("Hi YoonSung");
        } else {
            System.out.println("Who are you?");
        }
    }
```

```
Exception in thread "main" java.lang.NullPointerException
	at Practice.equalsTest1(Practice.java:6)
	at Practice.main(Practice.java:26)
```

* null 이 들어올시, `Who are you?` 가 아닌 NullPointerException이 발생한다.

---

## 상수를 먼저 씀으로써 회피한다.
> `상수.equals("변수")`

* eqauls함수에서 null 체크를 먼저 해버리므로, 예외를 회피할 수 있다.

```java
    public static void equalsTest2(String name) {
        //if (name.equals("JYS")) {
        if ("JYS".equals(name)) {   // null이 처리가 되어 else로 보낸다.
            System.out.println("Hi YoonSung");
        } else {
            System.out.println("Who are you?");
        }
    }
```

---

## 전체 소스

```java
class Practice {
    public static void equalsTest1(String name) {
        try {
            if (name.equals("JYS")) {
                System.out.println("Hi YoonSung");
            } else {
                System.out.println("Who are you?");
            }
        } catch (Exception e) {
            System.out.println(e.getMessage());
        }
    }

    public static void equalsTest2(String name) {
        if ("JYS".equals(name)) {
            System.out.println("Hi YoonSung");
        } else {
            System.out.println("Who are you?");
        }
    }

    public static void main(String[] args) {
        String name = "JYS";
        String nullName = null;

        equalsTest1(name);
        equalsTest1(nullName);

        equalsTest2(name);
        equalsTest2(nullName);
    }
}
```

```console
Hi YoonSung
null
Hi YoonSung
Who are you?
```

* 아주 사소한 차이지만, 결과는 사소하지 않을 수 있다.
* 방어적인 코딩을 하는 습관을 몸에 배여놓자.