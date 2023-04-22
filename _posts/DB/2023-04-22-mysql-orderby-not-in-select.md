---
title: ORDER BY clause is not in SELECT list
date: 2023-04-22-15:00
categories:
- DB

tags:
- DB
- DataBase
- MYSQL

---

## select 절에 없는 컬럼으로 정렬하려고 할떄 발생하는 에러를 알아봅니다.
> Expression #1 of ORDER BY clause is not in SELECT list

---

## Goal
- Expression #1 of ORDER BY clause is not in SELECT list 에러 원일을 알아봅니다.
- 설정으로 해결하는 방법을 알아봅니다.

---

## 에러의 원인
> Expression #1 of ORDER BY clause is not in SELECT list

- 기본적으로 **MYSQL** 에서 select 절에 없는것으로 정렬을 하려할떄 발생하는 에러입니다.
- `mysql 5.7` 버전 이상인 경우 `ONLY_FULL_GROUP_BY` 이 sql mode 에 삽입되어 있습니다.
- 이 옵션이 켜져있으면, select 절에 있는 컬럼으로만 정렬이 가능하게 됩니다. 

```sql
select distinct A.name, B.nickName
from A 
inner join a.id on b.a_id
orderBy A.id -- A.id 는 select 에 없어서 문제 발생.
limit 5
```

- 이때문에 예시의 경우 orderBy 조건으로 `A.id` 를 주었으나, select 절에 없으므로 에러가 발생하게 됩니다.

### 해결방법 
> `ONLY_FULL_GROUP_BY` 옵션을 비활성해줄 수 있습니다.

- `/etc/mysql/mysql.conf.d/mysqld.cnf`, **mysqld** 블록 아래에 `sql-mode=""` 를 넣어주면 됩니다.


---

## 주의할점
- 하지만 직접 DB에 raw Query 를 보내는경우에는 수행이되나, application 단에서는 에러가 계속 발생할 수 있습니다. `(example: jpa nativeQuery or QueryDSL)`
- 이유는 application 에서 사용하는 라이브러리나 프레임워크를 이용해서 db 에 쿼리를 보낼떄, 드라이버단에서 쿼리 실행을 막을 가능성이 존재하기 때문입니다.

---

## Conclusion
- mysql 5.7 이상부터는 디폴트 sql-mode 의 경우에는 select 절에 없는 컬럼으로 정렬조건을 줄 수 없다. (성능 이슈로 인해)
- sql-mode 를 변경해주면 정렬이 가능해진다.
- 사용하는 라이브러리나, 프레임워크가 있는경우 해당 드라이버단에서 쿼리 요청을 막을수도 있다.

---

## Reference
- https://stackoverflow.com/a/39353160
- https://stackoverflow.com/questions/41465332/incompatibility-with-mysql-5-7expression-1-of-order-by-clause-is-not-in-select