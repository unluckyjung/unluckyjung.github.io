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
> compare 오버라이딩을 이용해 정렬 기준이 2개인 경우를 정렬해봅니다.

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

## sort를 쓰기전에 정렬의 기준을 정해주어야한다.
> comparator 인터페이스를 이용하여 `compare`를 재정의 해준다.

- 왼쪽이 앞으로 가고 싶다면 `-1`
- 왼쪽이 뒤로 가고 싶다면 `1`
- 둘이 같다면 `0`
- 을 리턴한다.


- `c++`의 경우 왼쪽으로 간다면 **true**, 아니라면 **false** 을 리턴 했던것과는 반대의 상황이다.
  - `0` 도 추가되어있다.

```c++
// c++의 경우
bool cmp(const pair<int,int>& a, const pair<int,int>& b) {
	if (a.second == b.second) return a.first < b.first;
	return a.second < b.second;
}
```

---

## 사용자 정의 정렬을 해보자.
> 형이면 뒤로 가는 정렬을 해보자.

- 돈 많으면 형이다.
- 돈이 같다면 나이로 따진다.
  - 즉 21살 200만원 소유자가 22살 100만원 소유자보다 형이니 뒤로간다.

```java
package PS;

import java.util.*;

public class Main {
    public static void main(String[] args) {
        List<Person> persons = new ArrayList(Arrays.asList(new Person(21, 200), new Person(22, 100), new Person(23, 200)));
        Collections.sort(persons, new Comparator<Person>() {
            @Override
            public int compare(Person o1, Person o2) {
                if (o1.getMoney() == o2.getMoney()) {
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

## Collection sort시 List의 경우, Array.sort로 치환되어 돌아간다.
> 즉 아래와 같이 바꿀 수 도 있다.

- `Collection.sort` 를 `person.sort` 로 바꾸었다.

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

---

## Reference
- https://docs.oracle.com/javase/8/docs/api/java/util/Comparator.html