---
title: Java에서 private 메소드를 테스트하기
date: 2021-02-25-11:00
categories:
- TestCode

tags:
- Java
- AssetJ
- TestCode

---

## 자바에서 private 메소드를 테스트해봅니다.
> Junit5, AssetJ 를 이용해 테스트 합니다.

---

## priavte 메소드를 테스트하기

- [제일 깔끔하게 설명된 블로그](https://www.crocus.co.kr/1665)


### 테스트 하려는 메소드

```java
private int sum(int a, int b){
    return a + b;
}
```

### How To
- 해당 `private` 메소드를 테스트하려고 한다. 어떻게 할까?

- private 를 가진 클래스를 인스턴스화 해준다.
  - `Class instanceName = new Class();`
  
- 사용하고자 하는 private 메소드를 위해 Method 인스턴스를 만들어준다.
  - `Method method = 객체이름.getClass().getDeclared("private 메소드명", private 메소드에서 요구하는 파라메터)`
  - 즉 sum 메소드는 은 a,b 두개의 인자를 요구하므로
  - `Method method = instanceName.getClass().getDeclared("sum", a, b)`
  - 여기서 메소드의 이름은 `""` 로 감싸주는것을 놓치지 말자.
  
- 이제 Method 객체안에 담겨있는 `private sum` 메소드를 접근 허용 가능하게 해준다.
  - `method.setAccessible(true);`
  
- 마지막으로, `invoke` 를 통해 `private sum` 메소드를 실행시킨다. **(invoke 는 호출이라는 뜻)**
  - `(메소드 리턴타입)method.invoke(private 메소드를 가진 인스턴스명, private 메소드에서 요구하는 실제 파라메터 값)`
  - `int result = (int)method.invoke(instanceName, 1, 2)`
  - 여기서 인스턴스 이름은 `""` 이 없음에 주의 한다.

---

## 예제

```java
public class Regex {
    private static final Pattern IS_ONLY_NUMBER = Pattern.compile("^[0-9]*?");

    private boolean isOnlyNumber(final String str) {
        return IS_ONLY_NUMBER.matcher(str).matches();
    }
}
```

```java
    @Test
    @DisplayName("123-456-789 이 0과 양의 정수로만 이루어졌으면 true를 리턴한다.")
    void checkGetOnlyNumbers() throws Exception {

        Regex regex = new Regex();
        String str = "123-456-789";

        Method method = regex.getClass().getDeclaredMethod("isOnlyNumber", String.class);
        method.setAccessible(true);

        boolean result = (boolean) method.invoke(regex, str);
        assertThat(result).isEqualTo(false);
    }
```

- 문자열이 숫자로만 있는지 확인하는 private 메소드의 기능이 잘 작동하는지 테스트한다.

---

### 과연 위의 방법들을 통해서라도 private 메소드를 테스트 해야할까?
- 과연 private 메소드를 테스트 해야할까? : https://www.slipp.net/questions/253
- 그냥 private 메소드는 테스트 하지마라 라는 의견 : https://justinchronicles.net/ko/2014/07/14/dont-do-testing-private-methods/

- **개인적인 생각**으로는, 직접적으로 테스트를 하지 않는것이 맞다고 생각한다.
  - public 메소드에 연결된 경우를 통해서 테스트를 해야하지, 굳이 배보다 배꼽이 큰 테스트는 잘못되었다고 생각한다.
  - 그래도 알아는 두고 있자!

---

## Reference
- https://cocodo.tistory.com/16
- https://unabated.tistory.com/entry/jUnit-%EC%9C%BC%EB%A1%9C-Private-Method-%ED%85%8C%EC%8A%A4%ED%8A%B8-%EB%A7%8C%EB%93%A4%EA%B8%B0