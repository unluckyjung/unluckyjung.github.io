---
title: Spring 쿼리 파라미터 인자값 로그로 확인하기
date: 2022-01-28-20:30
categories:
- Spring

tags:
- Spring
- Database
- Log

---

## 쿼리 파라미터 인자값 로그로 확인하기
> 실제값이 아닌 `?` 로 로깅되는 쿼리 파라미터 값의 실제 값을 확인해봅니다.


## Goal
```sql
Hibernate: 
    insert 
    into
        member
        (id, name) 
    values
        (null, ?)
```

- 위와같이 로그가 나오는 상황에서 `?`의 실제값을 확인하는 법을 알아봅니다.

---

## yml 추가

```yml
logging.level:
  org.hibernate.type: trace
```

![image](https://user-images.githubusercontent.com/43930419/151162746-707d2f79-41fe-461c-bbc1-62225c95d48b.png)

- binding parameter 를 보고 `?`에 **jys**가 들어갔음을 확인할 수 있습니다.


## 외부 라이브러리 추가
> **[p6spy](https://github.com/p6spy/p6spy)** 를 사용합니다.

> P6Spy is a framework that enables database data to be seamlessly intercepted and logged with no code changes to existing application. The P6Spy distribution includes P6Log, an application which logs all JDBC transactions for any Java application.

- `p6spy`를 이용해 JDBC 트랜잭션을 전부 로그로 확인할 수 있습니다.

```java
dependencies {
    ...
	// 쿼리파라미터 로그확인용 외부 라이브러리
	implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.8.0'
    ...
}
```


> 라이브러리 [Github URL](https://github.com/gavlyukovskiy/spring-boot-data-source-decorator)

![image](https://user-images.githubusercontent.com/43930419/151162812-e67af356-6ba3-480d-934d-9c46905fd2f6.png)

```java
insert into member(id, name) values (null, 'jys')
```

- 좀더 명시적 으로 확인이 가능해진것을 볼 수 있습니다. (사진을 클릭하면 커집니다)
- **주의: 로깅작업이 어플리케이션 성능에 영향을 줄 수 있으므로 실제 배포시에도 사용하려면 성능측정이 병행되는것이 좋습니다.**

---

## Reference
- [SpringDocs](https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#server-properties)
- [https://github.com/gavlyukovskiy/spring-boot-data-source-decorator](https://github.com/gavlyukovskiy/spring-boot-data-source-decorator)
- [p6spy](https://github.com/p6spy/p6spy)
- [김영한 JPA 활용1](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-%ED%99%9C%EC%9A%A9-1)