---
title: 상속이 캡슐화를 깬다?
date: 2021-03-17-01:00
categories:
- OOP

tags:
- OOP
- Inheritance
- Encapsulation
- Effective Java

---

## 상속이 캡슐화를 꺤다는 말은 와닿지가 않는다.
> 상속으로 인해 캡슐화가 깨질 수 는 있지만, 깬다?

### Goal
- 캡슐화의 개념에 대해서 알아본다.
- 상속이 캡슐화를 깨는지 확인해본다.

---

## 캡슐화란?
> 외부에서 내부의 정보를 들여다 보지 못하게 하는것.

- 행위와 속성을 묶는다.
  - 객체지향 세계에서는 **객체가 행위를 할 수 있다.**
  - 실제 세계의 Cards가 할수 있는것은 아무것도 없다. 그냥 단순한 카드의 묶음일 뿐이다.
  - 하지만, 객체지향 세계의 Cards는 카드를 섞을수도 있고, 버릴수도 있다.
- 이런 행위와 속성을 하나의 묶음으로 가지고 있는것을 캡슐화라고 할 수 있다.

- 은닉화
  - 캡슐화와 은닉화는 다르다. 은닉화는 캡슐화로 인해서 얻어진 효과이다.
  - 객체를 사용하는 외부에서는, 객체가 어떻게 구성되어있는지 정확히 알 필요도 없고, 알 필요도 없다.
  - 객체에서 제공하는 메소드를 통해서 객체에게 메시지를 보내거나, 결과를 받아올뿐 그 과정이 어떻게 되는지는 모른다.

---

## 상속이 캡슐을 깨는 경우.

### 오버라이딩을 통해 메소드 재선언을 하는경우

```java
public class Car {
    public void move() {
        System.out.println("달린다.");
    }
}

class SuperCar extends Car {
    @Override
    public void move() {
        System.out.println("빨리 달린다.");
    }
}
```

- 다음과 같은 경우를 보자.
- `SuperCar`는 `Car`의 move 메소드를 오버라이딩하여 새로운 기능을 제공하고 있다.
- 이런경우, 부모 Car는 캡슐화가 유지되고 있지 못하게 된다.
  - 왜냐하면 외부에서 부모 Car의 move를 가져다 쓰기만 해야하는데, 재정의를 하고 바꾸어 쓰게 때문에 은닉화의 개념에서 **캡슐화가 깨졌다고 볼 수 있다.**
  - 상위 클래스의 구현이 하위 클래스에 노출되는것과 마찬가지이기 때문!

---

## 상속자체가 캡슐화를 깬다?

- 위의 예시를 보면 상속이 이루어지고, 그 뒤에 **오버라이딩 이라는 행위** 때문에 캡슐화가 깨졌음을 볼 수 있다.
- 하지만 여전히 상속자체가 캡슐화를 깬다는것은 이해가 가지않는다.
- 이 의문을 해결하기 위해서 구글링을 해보았다.

- [스택 오버플로우 링크](https://stackoverflow.com/a/40321880)에서 와닿는 문구를 찾았다.

> The word "break" might be too strong, but inheritance certainly does at least compromise encapsulation.

- 상속자체를 하는 행위 자체가 캡슐화를 망가뜨릴 수 있는 **가능성** 이 생기는거고
- 즉 상속이 캡슐화를 깬다 는 좀 과하지만 **캡슐화에 손상을 준다!** 라고 하면 좀 더 맞는거 같다.

---

## 이펙티브 자바 Item18
> 상속보다는 컴포지션을 사용하라.

![image](https://user-images.githubusercontent.com/43930419/111338050-f4427580-86b9-11eb-9be7-09a3481dc8b1.png)

- 대부분의 블로그 글에서 상속은 캡슐화를 깬다! 라고 써두고, 설명글을 보면 죄다 `상속 -> 오버라이딩`을 통해 상속이 깨지는 설명이 적혀 있었다.
  - 이것은 상속을 통해 오버라이딩이 캡슐화를 깨는것이다.
  - 상속 자체가 왜 캡슐화를 깨게 되는지에 대한 설명을 적어둔 블로그는 내 검색능력의 부족인지 찾지 못했다.

<br>

- 이런 고민을 하게 된 계기인 `이펙티브 자바 item18` 의 원문을 찾아보니.. 

![image](https://user-images.githubusercontent.com/43930419/111027456-cea43a80-8433-11eb-9f14-f71dfd72d644.png)

> but it is problematic because it violates encapsulation.  

- 캡슐화를 **위반한다** 라고만 써있지 **깬다** 라고처럼 강하게 써있진 않았다.

---

## 상속을 그러면 언제쓸까?
> 기능의 정제가 필요할때 생각해보자.

- 클래스 **상속**은 기능 재사용에 있어서 굉장히 강력한 도구인것이 사실이다.
- 하지만 생각보다 문제점이 많기 때문에, 클래스 상속보다는 컴포지션을 주로 이용하곤 한다.

- 클래스 상속보다는, 추상메소드나 인터페이스 상속을 통해 역할과 책임을 정제하는 용도로 사용하는것을 고려해보자.

---

## 제임스 고슬링의 코멘트
> 자바 언어 개발자, 자바의 아버지.. 제임스 고슬링이 말씀하시길

![image](https://user-images.githubusercontent.com/43930419/111027470-e5e32800-8433-11eb-9fab-69cc21955584.png)

<br>

![image](https://user-images.githubusercontent.com/43930419/111027481-fabfbb80-8433-11eb-9f3b-55c8d00bdf65.png)

> he explained that the real problem wasn't classes per se, but rather implementation inheritance (the extends relationship).   

- extends 상속은 문제가 있다고 말했다.

> You should avoid implementation inheritance whenever possible.  

- 가능하면 구현된것의 상속보다는, implement 상속을 고려해보라고 말했다. **즉 이미 구현된 class의 상속을 지양하라는 말이다.**

---

## Reference
- https://ko.wikipedia.org/wiki/%EC%BA%A1%EC%8A%90%ED%99%94
- https://www.infoworld.com/article/2073649/why-extends-is-evil.html
- https://www.infoworld.com/article/2076012/a-conversation-with-james-gosling.html