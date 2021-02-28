---
title: 제너릭에서 다형성을 통한 자식참조가 불가능한 이유
date: 2021-02-28-13:30
categories:
- Java

tags:
- Java
- Generic

---

## 부모 타입에 자식 타입을 참조시킬 수 없다.
> `List<Animals> animals = ArrayList<Dog>();` 은 불가능하다.

---

## 다형성을 이용해 참조시킬 수 있을까?
> Dog은 Animal의 자손 타입이니 가능하지 않을까?

```java
public class Animal {}

class Dog extends Animal {}

class Main {
    public static void main(String[] args) {
        List<Animal> animals = new ArrayList<Dog>();    // 에러
    }
}
```

- 해당 코드가 컴파일이 될거 같은가?
- 결론부터 이야기하면 **컴파일 에러가 발생한다.**
- 자바 제너릭을 처음 배울때, 보통 책에서는 타입변수와 실제타입이 일치해야만 생성이 가능하다고 설명해둔다.
  - 그냥 그대로 받아들이고 넘어갈 수 도 있지만, 
  - 왜 그런거지? 자손타입이면 가능해야 하는게 아닌가? 라는 생각을 한번 해볼 수 있지 않을까?

## 왜 안돼?
> 그러면 Animal List에는 Dog을 넣을수 없다는거야?

```java
public class Animal {}

class Dog extends Animal {}

class Main {
    public static void main(String[] args) {
        List<Animal> animals = new ArrayList<>();
        animals.add(new Dog());
    }
}
```

- 그렇다면 해당 코드도 안되는건가?
- 이것은 **된다.**

### 아 그러면 생성때만 안되는건가?
> Animal을 미리 만들어두고, Dog를 참조 시켜볼까?

```java
public class Animal {}

class Dog extends Animal {}

class Main {
    public static void main(String[] args) {
        List<Animal> animals;
        List<Dog> dogs = new ArrayList<>();
        animals = dogs;     // 에러
    }
}
```

- 역시나 **안된다.**

---

## 개들의 집합은 동물들의 집합이지 않은가?
> 하위집단으로는 가능하나, 참조가 될순 없다.

![image](https://user-images.githubusercontent.com/43930419/109407731-a265d400-79c6-11eb-9c87-3bf580d3fd98.png)

- 만약 Cat 이라는 Animal의 자손 클래스가 하나 더 있다고 생각해보자.

```java
public class Animal {}

class Dog extends Animal {}

class Cat extends Animal {}

class Main {
    public static void main(String[] args) {
        List<Animal> animals = new ArrayList<Dog>();    // 만약 된다면?
    }
}
```

- 만약 위에서 본 이 상황이 컴파일이 된다면?
- animal List에 Dog들이 들어가는것이 아닌, **animal List가 Dog List으로 만들어진다는 것이다.**

```java
List<Animals> animals = new ArrayList<Dog>();
animals.add(new Cat());
```

- 그렇다면 이 `Cat`은 어떻게 되는가?
- Cat이 의도했던것은 animals의 집합에 속하는것이다.
- 하지만 지금 animals는 Dog의 집합이 되어있다.
  - 그렇다면 **고양이는 개가 되는것인가?** 말이 안되는 소리이다. 

---

## 그렇다면 어떻게 고양이와 개를 넣을래?
```java
public class Animal {}

class Cat extends Animal {}

class Dog extends Animal {}

class Main {
    public static void main(String[] args) {
        List<Animal> animals = new ArrayList<>();
        List<Dog> dogs = new ArrayList<>(Arrays.asList(new Dog(), new Dog()));

        animals.addAll(dogs);

        for (Dog dog : dogs) {
            animals.add(dog);
        }

        animals.add(new Cat());
    }
}
```
- 다음과 같이 하면된다.


## 메소드로 분리하는 방법

```java
public class Animal {}

class Dog extends Animal {}

class Main {
    private static final List<Animal> animals = new ArrayList<>();

    public static void main(String[] args) {
        List<Dog> dogs = new ArrayList<>();
        addAnimals(dogs);   // 컴파일 에러가 발생한다.
    }

    private static void addAnimals(List<Animal> inputAnimals) {
        animals.addAll(inputAnimals);
    }
}
```

![image](https://user-images.githubusercontent.com/43930419/109408402-bdd3dd80-79cc-11eb-8e0f-505cf00d9421.png)

- 이글을 쓰게된 주 원인중 하나이다.
- 이제 왜 인자로 dogs을 넣을수 없는지를 알수 있을 것이다.

<br>

```java
public class Animal {}

class Cat extends Animal {}

class Dog extends Animal {}

class Main {
    private static final List<Animal> animals = new ArrayList<>();
    public static void main(String[] args) {
        List<Dog> dogs = new ArrayList<>(Arrays.asList(new Dog(), new Dog()));
        addAnimals(dogs);
        animals.add(new Cat());
    }

    private static void addAnimals(List<? extends Animal> inputAnimals){
        animals.addAll(inputAnimals);
    }
}
```
- 메소드로 분리하고 싶다면, 와일드 카드를 이용해 `method(List<? extends Animal> animal)` 매개변수로 넘겨줄 수 있다.
- **참조형으로 다형성을 이용할 수 없는것**이지, 해당 타입을 다형성으로 소화하지 못한다는 이야기가 아니기 때문이다.

---

## 자동으로 처리해주는것의 이유를 생각해보자

```java
List<Dog> dogs = new ArrayList<Dog>();
List<Dog> dogs = new ArrayList<>();
```

- 인텔리제이에서 첫번째 라인처럼 쓰면 안의 `Dog`을 굳이 써주지 않아도 된다고 이야기해준다.
- 아래처럼 쓰면 암시적으로 저안에 Dog을 넣어주기 때문이다.
  - 그 이유는 제너릭 타입으로 들어갈 수 있는것은 Dog 단 하나이기 때문이다!

---

## Reference
- https://stackoverflow.com/questions/5763750/why-we-cant-do-listparent-mylist-arraylistchild?noredirect=1&lq=1
- https://stackoverflow.com/questions/2745265/is-listdog-a-subclass-of-listanimal-why-are-java-generics-not-implicitly-po