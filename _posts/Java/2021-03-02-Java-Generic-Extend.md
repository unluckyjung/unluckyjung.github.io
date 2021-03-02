---
title: Java 제네릭 클래스 상속시 주의할점
date: 2021-03-02-12:20
categories:
- Java

tags:
- Java
- Generic

---

## 상속시 제네릭 타입도 같이 상속되었는지 확인해야 한다.
> 잘못된 상속이 이루어질 수 있다.

- 해당 주제는 같은 팀의 크루인[코기](https://ecsimsw.tistory.com/entry/%ED%97%B7%EA%B0%88%EB%A0%B8%EB%8D%98-%EC%A0%9C%EB%84%A4%EB%A6%AD-%ED%81%B4%EB%9E%98%EC%8A%A4-%EC%83%81%EC%86%8D-%EC%A0%95%EB%A6%AC)가 정리해준 내용을 다시 제 목소리로 재해석했습니다.
  - 외쳐 갓코기

---

## 오해
```java
public class MyClass{}

class Parent<T> {
    public T print(T arg) {
        System.out.println("1");
        return arg;
    }
}

class Child<T extends MyClass> extends Parent {
    public T print(T arg) {
        System.out.println("2");
        return arg;
    }
}

class App {
    public static void main(String[] args) {
        Parent<MyClass> p1 = new Parent<>();
        Parent<MyClass> p2 = new Child<>();
        Child<MyClass> c1 = new Child<>();

        p1.print(new MyClass());
        p2.print(new MyClass());
        c1.print(new MyClass()); 
    }
}
```

- 이 코드의 결과값은 무엇이 나올것 같은가?
- p1은 자명하게 1이 나올것이다.
- c1역시 자명하게 2가 나올것이다.
- 문제는 p2에서 발생한다.
  - 한번 보면, Child 클래스는 Parent를 상속했다.
  - `public T print(T arg)` 는 오버라이딩 될것을 기대한다.

```java
c1.print(new MyClass()); 
```
- 해당 메소드의 결과는 2를 기대한다.
- 하지면 결과는 **1**이 나오게된다.

---

## 왜?
> 1이 불렸다는것은 Parent의 메소드가 불렸다는건데? 어째서?

- 뜨거운 감자가 된 p2만 남기고 다 지워보자.

```java
public class MyClass{}

class Parent<T> {
    public T print(T arg) {
        System.out.println("1");
        return arg;
    }
}

class Child<T extends MyClass> extends Parent {
    public T print(T arg) {
        System.out.println("2");
        return arg;
    }
}

class App {
    public static void main(String[] args) {
        Parent<MyClass> p2 = new Child<>();
        p2.print(new MyClass());
    }
}
```

![image](https://user-images.githubusercontent.com/43930419/109493518-f90b0500-7acf-11eb-8443-78d7f4b1043c.png)


- 우리의 똑똑한 인텔리제이는 Child의 print는 아예 사용되지 않고 있다고 이야기해준다.
- 어째서 이런일이 발생하게 된것일까?

### 잘못된 상속
- 근본적인 이유는 상속이 잘못 이루어졌기 때문이다.
- 우리는 Parent를 상속할때 제네릭타입을 같이 가져오지 않았다.

```java

// 잘못된 상속
class Child<T extends MyClass> extends Parent {
    public T print(T arg) {
        System.out.println("2");
        return arg;
    }
}

// 올바른 상속
class Child<T extends MyClass> extends Parent<T> {
    public T print(T arg) {
        System.out.println("2");
        return arg;
    }
}
```

- 아래와 같이 **올바른 상속**을 하게 된다면, 정상적으로 오버라이딩이 되어 1이 아닌 **2**의 결과를 얻을 수 있다.

```java
public class MyClass{}

class Parent<T> {
    public T print(T arg) {
        System.out.println("1");
        return arg;
    }
}

class Child<T extends MyClass> extends Parent<T>{
    public T print(T arg) {
        System.out.println("2");
        return arg;
    }
}

class App {
    public static void main(String[] args) {
        Parent<MyClass> c1 = new Child<>();
        c1.print(new MyClass());    //  2가 호출된다.
    }
}
```

---

## 아니 그러면 처음 컴파일 타임에서부터 에러가 났어야 했던거 아니야?
> 잘못된 상속이라매, `Parent<MyClass> c1 = new Child<>();` 이게 그러면 불가능 해야 하는거아니야?

- 이어 설명하기 전에 확실히 짚고 넘어갈 부분이 있다.
  - **잘못된 상속** 이라고 했지, **상속이 되지 않았다** 라고 하지 않았다.
  - **잘못된 상속** 의 정의가 좀 모호하긴 하지만, 애초에 맨 위의 방식으로 짜는것 자체가 잘못된 방식이기때문에, 그냥 잘못된 상속이라고 하고 넘어가겠다.
  - 지금 우리가 궁금한것은, 왜 이런일이 벌어졌냐지 다른곳에 촛점을 맞추진 말자


- 다시 잘못된 코드로 돌아가보자.
  - 위로 올려서 보기 귀찮을것 같으므로 친절하게 다시 복붙해주겠다.


```java
public class MyClass{}

class Parent<T> {
    public T print(T arg) {
        System.out.println("1");
        return arg;
    }
}

class Child<T extends MyClass> extends Parent{
    public T print(T arg) {
        System.out.println("2");
        return arg;
    }
}

class App {
    public static void main(String[] args) {
        Parent<MyClass> p2 = new Child<>();
        p2.print(new MyClass());
    }
}
```

- 여기서 이야기 해주고 싶은것은 print는 전혀 다른 print이다.
  - 즉 Parent 클래스의 print와 Child 클래스의 print는 이름이 같아 헷갈린것이다.
- 왜 달라? 
  - `public T print(T arg)` 에는 타입변수인 T가 있다.
  - 이 `T`의 경우 Parent print의 T와, Child print의 T는 다르다.
  - 또 왜?
    - 잘못된 상속을 했기 때문이다.
    - Parent에 대한 상속만 했지, 제네릭에 대한 상속은 이루어지지 않았기 때문이다.
    - [이 경우](#잘못된-상속)를 다시 보고 오자.



### 그래서 어떻게 상속이 된건데?
- 계속 혼란을 주고 있는 메소드명만 바꿔서 보면 직관적으로 확인이 가능하다.

```java
public class MyClass{}

class Parent<T> {
    public T parentPrint(T arg) {
        System.out.println("1");
        return arg;
    }
}

class Child<T extends MyClass> extends Parent{
    public T childPrint(T arg) {
        System.out.println("2");
        return arg;
    }
}

class App {
    public static void main(String[] args) {
        Parent<MyClass> p1 = new Child<>();
        Child<MyClass> c1 = new Child<>();
    }
}
```

![image](https://user-images.githubusercontent.com/43930419/109495561-cca4b800-7ad2-11eb-8af7-0cb146dc9de9.png)

![image](https://user-images.githubusercontent.com/43930419/109495722-ff4eb080-7ad2-11eb-9b1f-fde2051afeed.png)


- p1의 경우, Parent에만 구현되어있는 ParentPrint 메소드에만 접근이 가능한걸 볼 수 있다.
- c1의 경우, 두가지 메소드에 다 접근이 가능한것을 볼 수 있다.


<br><br>

![image](https://user-images.githubusercontent.com/43930419/109495849-2dcc8b80-7ad3-11eb-811f-df8b28d3483c.png)

- 즉 메소드 이름을 다시 동일하게 print로 바꾼경우, 다음과 같이 두개가 다른 메소드로 표현되고 있음을 알 수 있다.

---

## 결론
> 제네릭 타입을 가지고 있는 클래스를 상속할때는 주의하자.

- 제네릭 타입을 상속하지 않아도 물론 컴파일 에러는 나지 않는다.
- 하지만 대부분의 상황에서 Parent 만 상속하는것이 아니고 `Parent<T>` 를 상속하는것이 아마 의도에 맞을것이다.

---

## 생각해볼거리

### 1

```java
package generic_lecture;

class Parent<T extends Number> {
    public void print(T arg) {
        System.out.println(arg);
    }
}

class Child<T> extends Parent{
    public void print(T arg) {
        System.out.println(arg);
    }
}

class App {
    public static void main(String[] args) {
        Child c1 = new Child();
        c1.print(1234);
        c1.print("Goodgid ManSae");
    }
}
```

- 다음 코드의 결과는 어떻게 될것 같은가?
- 컴파일 에러가 날것인가?
- 아니면 문자열을 출력하는 시점에서 런타임 에러가 날것인가?
- 또 아니면 숫자와 문자열이 다 정상적으로 출력이 될것인가?


### 2

```java
class Parent<T extends Number> {
    public void print(T arg) {
        System.out.println(arg);
    }
}

class Child<T> extends Parent<T>{
    public void print(T arg) {
        System.out.println(arg);
    }
}

class App {
    public static void main(String[] args) {
        Child c1 = new Child();
        c1.print(1234);
        //c1.print("Goodgid ManSae");
    }
}
```

- [1](#1)이 컴파일 단에서 문제가 있었을거라고 생각해보자.
- 그렇다면 child 부분을 다음과 같이 변경하고 문자열을 주석 처리한다면 컴파일이 정상적으로 될까?
- 안된다면 이유는 무엇일까?
- 답은 [이곳](https://gist.github.com/unluckyjung/222356008f3a4b0f212d1d81917aaeb7)의 댓글에 있습니다.

---

## Reference
- https://ecsimsw.tistory.com/entry/%ED%97%B7%EA%B0%88%EB%A0%B8%EB%8D%98-%EC%A0%9C%EB%84%A4%EB%A6%AD-%ED%81%B4%EB%9E%98%EC%8A%A4-%EC%83%81%EC%86%8D-%EC%A0%95%EB%A6%AC