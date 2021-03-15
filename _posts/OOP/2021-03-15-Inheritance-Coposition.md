---
title: 상속 대신 조합을 고려해보자
date: 2021-03-15-15:55
categories:
- OOP

tags:
- OOP
- Inheritance
- Composition

---

## 상속은 생각보다 단점이 많다.
> 상속으로 발생하는 Side effect를 조합으로 우회 해보자.

### Goal
- 상속에 대해 착각하고 있는점과 단점을 알아본다.
- 조합을 이용해 상속의 단점을 피해본다.
- 언제 상속을 쓸지, 조합을 쓸지를 알아본다.
  
---

## 상속에 대한 오해
> 중복된 부분을 재사용하는데 굉장히 용이하다! 상속은 참 좋다!

- 상속은 코드 재사용에 있어서 굉장히 강력한 무기이다. 
- 그렇다면 기능이 재사용 되는 부분에는 상속을 쓰는것이 항상 옳을까?

### 부작용 예시 1
> 예상치 못한 결과를 야기할 수 있다.

```java
public class MyHashSet<E> extends HashSet<E> {
    private int addCount;

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}

class Main {
    public static void main(String[] args) {
        MyHashSet<Integer> myHashSet = new MyHashSet<>();
        myHashSet.addAll(Arrays.asList(1, 2, 3));
        System.out.println(myHashSet.getAddCount());
    }
}
```

```
// result

6
```

- `어? 나는 3을 기대했는데?`
  - 이에 대한 근거는 addCount에 `c.size()`를 더할것이고, `c.size()` 의 기댓값은 3이기 때문이다.

```java
    @Override
    public boolean add(E e) {
        System.out.println("add called!");
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        System.out.println("addAll called!");
        addCount += c.size();
        return super.addAll(c);
    }
```

- 이유를 찾기 위해 로그를 찍어보자.

```
// result

addAll called!
add called!
add called!
add called!
```

- `addAll()`를 호출시, `add()`가 같이 불리는것을 확인할 수 있다.

<br>

![image](https://user-images.githubusercontent.com/43930419/110587985-b2946500-81b7-11eb-8082-abec02c3de5c.png)

- 실제 구현된 파트를 보면, `addlAll()` 은 `add()`를 호출하는 방법으로 구현되어있는데, 이부분도 오버라이딩이 되어 발생한 결과이다.

```java
    @Override
    public boolean addAll(Collection<? extends E> c) {
        //addCount += c.size();
        return super.addAll(c);
    }
```

> 그럼 이렇게 치워버리면 되는거 아니야?

- 이방법은 우리가 앞의 예시를 통해서, `add()` 가 호출된다는것을 **알게 된뒤에** 찾아낸 해결책이다.
  - 과연 개발을 하면서 이런 경우를 전부 일일히 찾아서 확인할 수 있겠는가?
  
- 기대하는 addAll()의 역할을 수행하는 다른 메소드를 구현할 수 도 있다.
  - 하지만 이방법은 이미 구현된 기능을 새로 만드는것이 되는데, 그것이 과연 좋은 행동일까?
  - 조금 핀트랑은 어긋나지만, 이미 잘 구현된 정렬알고리즘이 있음에도 불구하고, 프로그래머가 정렬 알고리즘을 다시 짜는 행위와 비슷하다.

---

## 부작용 예시2
> 매서드 해석이 모호할 수 있다.

```java
package oop;

public class Document {
    public int length() {
        return this.content().length();
    }

    public String content() {
        return "abc";
    }
}

class KoreanTranslateDocument extends Document {
    @Override
    public String content() {
        return "에이비씨";
    }
}

class Main {
    public static void main(String[] args) {
        Document document = new KoreanTranslateDocument();
        System.out.println(document.length());  // "에이비씨"의 길이? "abc"의 길이?
    }
}
```

```
// result

4
```

- `length()` 를 불렀을때 어느 문자열의 길이가 나올지 단번에 예상할 수 있었는가?
  - 예상이 가능했어도, 과연 모든 사람들이 그럴수 있을꺼라고 확신할 수 있는가?
  - 잠깐이라도 어? `"abc"` 의 길이가 나오는게 아니야 하고 생각이 들진 않았는가?

- 사실 이 관계는 모호하게 Document, KoreanTranslateDocument 라는 예시를 들어서 그렇지, Document를 `EnglishDocument` 이거나, `OriginalDocument`라고 생각하면 `IS-A` 관계에서 어긋난다는것을 쉽게 알아챌 수 있다.

```java
public abstract class Document {
    public int length() {
        return this.content().length();
    }

    public abstract String content();
}

class OriginalDocument extends Document {
    @Override
    public String content() {
        return "abc";
    }
}

class KoreanTranslateDocument extends Document {
    @Override
    public String content() {
        return "에이비씨";
    }
}
```

- 혼란을 낳게 되는 content() 부분을 `abstract` 로 구성해주어 상속한 클래스에서 구현하는것을 강제하게한다면
- 이제 `length()` 의 결과값을 모호하지 않게 단숨에 예상할 수 있게된다.
- 그리고 처음에 어긋낫던 `IS-A` 관계를 수정할 수 있는 인사이트도 생기게 되었고, 추상화를 좀 더 자연스럽게 할 수 있게 되었다.

---

## 부작용 예시3
> 상위 클래스에 굉장히 의존적이게 된다.


```java
public class Document {
    public int length() {
        return this.content().length();
    }

    //public String content() {  
    public char[] content() {   
        // String에서 char[]로 바뀌었다.
        // 해당 메소드를 이용하던 하위 클래스들은 많은 변경이 이루어져야 할것이다.
    }
}
```

- 만약 `content()`의 리턴값이 바뀐다고 생각해보자.
- Document의 하위 클래스에서 content()를 이용해 구현된 모든 사항은 변경이 이루어져야 할것이다.

---

## 부작용 예시4
> 오버라이딩을 통한 재정의는 캡술화를 손상시키게 된다.


```java
public class Car {
    public void run() {
        System.out.println("달린다.");
    }
}

class SuperCar extends Car {
    @Override
    public void run() {
        System.out.println("빨리 달린다.");
    }
}

```

- 캡슐화의 개념에 대해서 생각해보자.
  - 간단히 말하면 자기가 가지고 있는 내용을 남들에게 보여주지 않는것이다.
  - 메소드 자체가 어떻게 구현되어있는지를 노출하지 않는다.
  - 사용자는 메소드의 내용을 전혀 몰라도, 제공되는 메소드 이름만을 통해서 객체를 다룰수 있게 된다.
  
- 하지만 오버라이딩을 한다면? 기존에 목적했던 보호하는 역할, 숨기는 역할을 제대로 수행하게 되지 못하는것이다.
- 상속과 캡슐화에 대한 내용은 나중에 다른문서를 통해 자세히 정리해보려고 한다.
  - 상속자체가 캡슐화를 **깬다**?
  - 상속은 캡슐화를 **손상** 시킬 가능성이 있는것이지, **깬다** 라고 보기에는 힘들다.
  
- https://stackoverflow.com/questions/9344935/inheritance-breaking-encapsulation
- https://stackoverflow.com/questions/40321009/inhertitance-breaks-encapsulation


---

## 조합(Composition)을 사용해보자.
- 처음에 조합, 컴포지션? **무슨말이지?** 라는 생각이 들수 있다.
  - 나의 경우 알고리즘 문제 풀이에서 써먹는 `조합(Combination)` 이 먼저 떠올랐다...


- 왜 조합이라는 이름이 붙었냐하면...
  - 이미 만들어진 객체를 이용해 **조합**하여, 새로운 객체를 구성하기 때문이다.
  - 레고의 부품(기존에 만들어진 객체)를 조합해 붙인다고 생각하면 이해가 쉽다!


### 조합대신 상속을 사용하는 경우
> 아직 외계인인것을 모르는 슈퍼맨

```java
public class Man {
    public void move() {
        System.out.println("걷는다");
    }

    public void eat() {
        System.out.println("먹는다");
    }
}

class SuperMan extends Man {
    public void fly() {
        System.out.println("날아간다.");
    }
}
```

- 위와 같이 SuperMan이 Man을 상속하는 식으로 구성했다고 해보자.
- 나중에 SuperMan이 Man(사람)이 아닌것을 알게 되었다면?
  - 실제로 슈퍼맨은 인간이 아니고 외계인이다. 즉 `IS-A` 관계가 되지 않는다는것을 나중에 알게 된 경우이다.

```java
class Man {
    public boolean canTouchKryptonite(){
        return true;
    }
}
```

- 슈퍼맨은 크립토나이트를 만지지 못하는데, Man에 크립토나이트를 만지는 기능이 생긴다면?
    - 이걸 또 상속해서 재정의 하고 그래야될까?

```java
class SuperMan extends Man{
    @Override
    public boolean canTouchKryptonite(){
        return false;
    }
}
```

---

### 조합을 사용하는 경우

```java
public class Man {
    public void move() {
        System.out.println("걷는다");
    }

    public void eat() {
        System.out.println("먹는다");
    }
}

class SuperMan {
    private final Man man = new Man();

    public void move() {
        man.move();
    }

    public void eat() {
        man.eat();
    }

    public boolean canTouchKryptonite(){
        return false;
    }

    public void fly() {
        System.out.println("날아간다.");
    }
}
```

- 다음과 같이 구성을 해놓으면, Man 클래스의 변화에 대해서 의존적이지 않게된다.
  - 단순히, man의 메소드를 호출하는 방식으로 바뀌니까!

---

## 언제 상속? 언제 조합?
> 상속을 쓰지 말라는 이야기는 아니다.

- 우리는 처음 OOP 언어를 배울때 `상속은 정말 좋은것!`, `상속을 쓰는쪽으로 생각해봐!` 라는 생각을 주입? 받으며 배우게 된다.
- 하지만 생각보다 상속은 단점이 많다는것을 인지하는것이 중요하다.
- 다형성을 위해서는 인터페이스나, 추상 클래스를 이용한 구현을 고려해보자.
  - 상속을 기능의 재활용보다는, 정제를 위해 사용하자! 

- **이런 경우에는 상속을 고려해보자.**
  - 코드 재사용을 주목적으로 하기보다는 확장성, 유연성을 고려해야할때
  - `IS-A` 관계가 명확할때
  - 부모 메소드에 이미 구현된 내용이 절대 바뀌지 않는다고 확신이 들때

---

## Reference
- https://woowacourse.github.io/javable/post/2020-05-18-inheritance-vs-composition/
- https://madplay.github.io/post/favor-composition-over-inheritance
- https://jgrammer.tistory.com/entry/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C-%EC%9E%90%EB%B0%94-%EC%83%81%EC%86%8D%EB%B3%B4%EB%8B%A4%EB%8A%94-%EC%BB%B4%ED%8F%AC%EC%A7%80%EC%85%98%EC%9D%84-%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC