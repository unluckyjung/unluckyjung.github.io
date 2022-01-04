---
title: Java에서 사용자 정의 멀티 정렬하기
date: 2021-02-23-22:30
categories:
- Java

tags:
- Java
- Sort

---

## Java에서 사용자 정의 정렬을 해봅니다.
> `Comparator`를 이용해 정렬 기준이 2개인 객체를 정렬해봅니다.

---

## 정렬 하려는 대상
> age와 money를 가지고 있는 Person

```java
class Person {
    private int age;
    private int money;

    Person(final int age, final int money) {
        this.age = age;
        this.money = money;
    }

    public int getAge() {
        return age;
    }

    public int getMoney() {
        return money;
    }

    @Override
    public String toString() {
        return "Person{" +
                "age=" + age +
                ", money=" + money +
                '}';
    }
}
```

---

## sort를 쓰기전에 정렬의 기준을 정해주어야 한다.
> comparator compare 메소드를 구현하여 정렬기준을 정해줍니다.

- 앞으로 가고 싶다면 `-1`
- 뒤로 가고 싶다면 `1`
- 같다면 `0`을 반환한다.

- `c++`의 경우 선두로 간다면 **true**, 반대의 **false** 을 리턴 했던것과는 반대의 상황이다.
- 그리고 `0` 도 추가되어있다.

```c++
// a,b 값이 더 작다면 선두로 간다.
bool cmp(const pair<int,int>& a, const pair<int,int>& b) {
	if (a.second == b.second) return a.first < b.first;
	return a.second < b.second;
}
```

---

## Comparator를 이용해 사용자 정의 정렬을 해보자.
> 형이면 뒤로 가는 정렬을 해보자. (오름차순)

- 돈 많으면 형이다. 돈이 같다면 나이로 형을 정한다.

```
// 원하는 결과
{ (22, 100), (21, 200), (23, 200) }
```

- 21살은 나이가 더 어리지만 22살 100만원보다 돈이 더많아 형이된다.
- 21살 23살은 같은 금액을 보유하고 있으므로, 나이를 따지게 되어 23살이 형이된다.

```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        List<Person> persons = new ArrayList(Arrays.asList(new Person(21, 200), new Person(22, 100), new Person(23, 200)));
        Collections.sort(persons, new Comparator<Person>() {
            @Override
            public int compare(Person o1, Person o2) {
                // 가진 금액이 같다면 
                if (o1.getMoney() == o2.getMoney()) {
                    // 나이로 따진다.
                    return o1.getAge() < o2.getAge() ? -1 : o1.getAge() > o2.getAge() ? 1 : 0;
                } else if (o1.getMoney() < o2.getMoney()){
                    return -1;
                } else{
                    return 1;
                }
            }
        });

        System.out.println(persons.toString());
    }
}
```

```
// 결과

[Person{age=22, money=100}, Person{age=21, money=200}, Person{age=23, money=200}]
```

---

## List의 경우, 객체.sort로 바꿀수 있다. 
> 자바8 이상인 경우 람다식도 사용 가능하다.

```java
public class Main {
    public static void main(String[] args) {
        List<Person> persons = new ArrayList(Arrays.asList(new Person(21, 200), new Person(22, 100), new Person(23, 200)));
        persons.sort((o1, o2) -> {
            if (o1.getMoney() == o2.getMoney()) {
                return o1.getAge() < o2.getAge() ? -1 : o1.getAge() > o2.getAge() ? 1 : 0;
            } else if (o1.getMoney() < o2.getMoney()) {
                return -1;
            } else {
                return 1;
            }
        });

        System.out.println(persons.toString());
    }
}
```

---

## int 비교시 삼항 연산자를 제거할 수 있다.
> 삼항연산자를 `Integer.compare`로 치환할 수 있다.

- 기존의 `o1.getAge() < o2.getAge() ? -1 : o1.getAge() > o2.getAge() ? 1 : 0;` 를  
- `Integer.compare(o1.getAge(), o2.getAge())` 와 같이 바꿀수 있다.

```java
public class Main {
    public static void main(String[] args) {
        List<Person> persons = new ArrayList(Arrays.asList(new Person(21, 300), new Person(23, 200), new Person(22, 200)));
        persons.sort((o1, o2) -> {
            if (o1.getMoney() == o2.getMoney()) {
                return Integer.compare(o1.getAge(), o2.getAge());
            } else if (o1.getMoney() < o2.getMoney()) {
                return -1;
            } else {
                return 1;
            }
        });

        System.out.println(persons.toString());
    }
}
```

- `Comparator` 인터페이스 외에도 `Compable` 혹은 정적 메소드를 이용한 방법들이 존재한다.
- 다른 글에서 추가적으로 정리하도록 한다.

---

## Reference
- https://docs.oracle.com/javase/8/docs/api/java/util/Comparator.html