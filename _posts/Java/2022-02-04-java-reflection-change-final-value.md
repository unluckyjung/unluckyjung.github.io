---
title: Java final이 붙은 값을 reflection 을 통해 변경해보기
date: 2022-02-04-03:06
categories:
- Java

tags:
- Java
- Reflection

---

## final 키워드가 붙은 변수를 리플랙션을 통해 변경하는법을 알아봅니다.
> static final 키워드도 변경이 가능한지 확인해봅니다.

---

## Goal
- **final** 키워드가 붙은 변수를 리플랙션을 통해 변경하는법을 알아봅니다.
- **static final** 의 경우에도 변경이 가능한지 확인해봅니다.

---

## How

```java
클래스 인스턴스명 = new 클래스
Class clazz = 클래스명.class;
Field field = clazz.getDeclaredField("필드변수명");
field.setAccessible(true);
field.set(인스턴스명, "바뀔 필드값");
```

### 예시

```java
import org.junit.jupiter.api.Test;

import java.lang.reflect.Field;

import static org.assertj.core.api.Assertions.assertThat;

class FinalChangeTest {
    @Test
    void test() {
        UserName userName = new UserName("jys");

        try {
            Class clazz = UserName.class;
            Field field = clazz.getDeclaredField("value");
            field.setAccessible(true);

            // jys -> unluckyjung으로 변경
            field.set(userName, "unluckyjung");

            assertThat(userName.getValue()).isEqualTo("unluckyjung");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

class UserName {
    private final String value;

    UserName(final String value) {
        this.value = value;
    }

    public String getValue() {
        return value;
    }
}
```

```java
class UserName {
    private final String value;
    ...
}
```

- **jys**로 설정되었던 value값이 **unluckyjung**으로 변경되어 테스트를 통과하는것을 확인하실 수 있습니다.

<br>

## static final 도 가능합니다.
- **런타임 타임때** 설정된 `static final` 형태도 변경이 가능합니다. (하지만 이런 경우가 필요한 상황은 굉장히 드뭅니다) 

```java
import java.lang.reflect.*;

public class EverythingIsTrue {
   static void setFinalStatic(Field field, Object newValue) throws Exception {
      field.setAccessible(true);

      Field modifiersField = Field.class.getDeclaredField("modifiers");
      modifiersField.setAccessible(true);
      modifiersField.setInt(field, field.getModifiers() & ~Modifier.FINAL);

      field.set(null, newValue);
   }
   public static void main(String args[]) throws Exception {      
      setFinalStatic(Boolean.class.getField("FALSE"), true);

      System.out.format("Everything is %s", false); // "Everything is true"
   }
}
```

- [StackOverflow 에서 가져온 예시](https://stackoverflow.com/a/3301720/11355002)  
- **false**가 아니라, **true**가 출력되는것을 확인할 수 있습니다.

---

## Conclusion
- `final` 키워드가 붙은 변수값을 `reflection`을 통해 변경할 수 있다. 
- **런타임**때 설정되는 `static final` 값인 경우에도 변경이 가능하다.
- 하지만 final 이 붙은 값을 무작정 바꾸려고 하기보다는, **바꾸려는 상황이 맞는지, 설계에 문제가 있는것이 아닌지**를 먼저 고려해보자.


---

## Code 
- 해당 코드들은 전부 [github](https://github.com/unluckyjung/blog-codes/tree/main/reflection-change-final-value)에서 보실 수 있습니다.

---

## Reference
- [https://stackoverflow.com/questions/3301635/change-private-static-final-field-using-java-reflection/3301720#3301720](https://stackoverflow.com/questions/3301635/change-private-static-final-field-using-java-reflection/3301720#3301720)
- [https://stackoverflow.com/questions/17506329/java-final-field-compile-time-constant-expression](https://stackoverflow.com/questions/17506329/java-final-field-compile-time-constant-expression)