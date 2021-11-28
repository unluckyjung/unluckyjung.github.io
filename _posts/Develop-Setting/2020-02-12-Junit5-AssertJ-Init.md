---
title: 인텔리제이 Junit5, AssertJ 초기설정
date: 2021-02-12-20:57
categories:
- Develop-Setting

tags:
- Intellij
- Junit5
- AssertJ

---

## 인텔리제이에서 Junit5 AssertJ 초기설정을 해봅니다.
> intellij 버전 : 2020.3.1  

---

## Junit5 프로젝트 만들기
> [참고링크](https://docs.gradle.org/current/userguide/java_testing.html#using_junit5)

### intellij 에서 gradle 로 프로젝트를 만든다.

![img](/post_images/Junit5-0.png)

### bulid.gradle에 의존성을 추가해준다.

![img](/post_images/Junit5-1.png)

```
dependencies {
    testImplementation 'org.junit.jupiter:junit-jupiter-api:5.6.0'
    testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine'
}
```

- 해당 의존성들은 `intellij 2020.3.1 silicon` 버전에서 자동으로 추가된것들이다.

---

## AssertJ

### bulid.gradle에 의존성을 추가해준다.

- [공식 문서 퀵스타트](https://joel-costigliola.github.io/assertj/assertj-core-quick-start.html)

```java
testCompile 'org.assertj:assertj-core:3.11.1'
```

```java
dependencies {
    testImplementation 'org.junit.jupiter:junit-jupiter-api:5.6.0'
    testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine'
    testCompile 'org.assertj:assertj-core:3.11.1'
    // testImplementation 'org.assertj:assertj-core:3.11.1' 
    // implementation으로 해도 가능합니다. compile과 차이점은 추후 포스팅할예정.
}
```


### import 하기
- assertThat() import 는 `()` 까지 쳐준뒤 MAC 기준 `option + enter` 를 눌러주어야 한다.
  - 메소드만 만드라고 나오는경우, `assertThat(비교할 타겟)` 이렇게 타겟까지 넣어놓고, 커서를 이동하다가, 밑줄이 그어진순간 `option + enter` 를 시도해보자.
  - 그래도 안되면 `import static org.assertj.core.api.Assertions.*;` 를 직접 추가 해주자.

---

## Reference
- https://joel-costigliola.github.io/assertj/index.html
- https://itbellstone.tistory.com/106