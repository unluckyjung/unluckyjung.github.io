---
title: 프로그래머스 SQL 고득점 KIT [SELECT]
date: 2020-07-26-15:00
categories:
- PS
- SQL

tags:
- programmers
- SQL

---

## 프로그래머스 SQL [SELECT](https://programmers.co.kr/learn/courses/30/parts/17042)을 풀어봅니다.
> SELECT `7문제`를 한 포스팅에 담아봅니다.

---


## [모든 레코드 조회하기](https://programmers.co.kr/learn/courses/30/lessons/59034)

* `*` 에스터리크를 이용하여 테이블 정보를 모두 조회합니다.   `SELECT`
* `아이디` 순으로 정렬합니다.   `ORDER BY`

```sql
SELECT * FROM ANIMAL_INS 
ORDER BY ANIMAL_ID;
```

---

## [역순 정렬하기](https://programmers.co.kr/learn/courses/30/lessons/59035)

* `이름` 과 `보호 시작일` 을 조회합니다.    `SELECT`
* `아이디` 역순으로 정렬합니다.     `ORDER BY` , `DESC`

```sql
SELECT NAME, DATETIME FROM ANIMAL_INS 
ORDER BY ANIMAL_ID DESC
```

---

## [아픈 동물 찾기](https://programmers.co.kr/learn/courses/30/lessons/59036)

* `아이디` 와 `이름` 을 조회합니다.     `SELECT`
* `아픈 동물`만 고릅니다.   `WHERE`, `=`
* `아이디` 순으로 조회합니다.   `ORDER BY` 

```sql
SELECT ANIMAL_ID, NAME FROM ANIMAL_INS
WHERE INTAKE_CONDITION = "Sick"
ORDER BY ANIMAL_ID;
```

---
## [어린 동물 찾기](https://programmers.co.kr/learn/courses/30/lessons/59037)

* `아이디` 와 `이름` 을 조회합니다.     `SELECT`
* `나이가 많지 않은` 동물만 고릅니다.   `WHERE` ,`<>` , `!=`
* `아이디` 순으로 조회합니다.   `ORDER BY` 

```sql
SELECT ANIMAL_ID, NAME FROM ANIMAL_INS
-- WHERE INTAKE_CONDITION <> 'Aged'
WHERE INTAKE_CONDITION != "Aged"
ORDER BY ANIMAL_ID;
```

---

## [동물의 아이디와 이름](https://programmers.co.kr/learn/courses/30/lessons/59403)

* `아이디` 와 `이름` 을 조회합니다.     `SELECT`
* `아이디` 순으로 조회합니다.   `ORDER BY` 

```sql
SELECT ANIMAL_ID, NAME FROM ANIMAL_INS 
ORDER BY ANIMAL_ID
```

---

## [여러 기준으로 정렬하기](https://programmers.co.kr/learn/courses/30/lessons/59404)

* `아이디` 와 `이름` , `보호 시작일을` 조회합니다.     `SELECT`
* `이름` 순으로 조회합니다.   `ORDER BY` 
* `이름`이 같다면, `보호를` **나중에** 시작한 순으로 조회합니다.    `ORDER BY`, `DESC`

```sql
SELECT ANIMAL_ID, NAME, DATETIME FROM ANIMAL_INS
ORDER BY NAME, DATETIME DESC
```

---

## [상위 n개 레코드](https://programmers.co.kr/learn/courses/30/lessons/59405)

* `이름`을 조회합니다.     `SELECT`
* `들어온` 날짜 순으로 정렬합니다.      `ORDER BY`
* 가장 먼저 들어온 동물 `하나` 만 조회 합니다.      `LIMIT`

```sql
SELECT NAME FROM ANIMAL_INS
ORDER BY DATETIME
LIMIT 1
```

---

## 피드백
* 단순히 문제를 푸는것과, 남들에게 풀이를 설명하는 거랑은 많이 다르단걸 또 다시 느꼇다.
* 최근 개발을 많이 안해서 `SQL` 이 익숙치 않다.
* 알고리즘처럼 SQL 문제를 꾸준히 풀어서 안 잊어버리게 해야겠다.