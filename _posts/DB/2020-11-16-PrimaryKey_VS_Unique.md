---
title: 기본키와 Unique의 차이
date: 2020-11-16-17:00
categories:
- DB

tags:
- MYSQL
- DB

photos:
- /post_images/mysql.png

---

## PrimaryKey vs Unqiue
> 비슷하지만 다른 둘을 비교해봅니다.

---

## PrimaryKey

```sql
ALTER TABLE sampleTable ADD CONSTRAINT pkey PRIMARY KEY(colName);
```

---

## Unique, NOT NULL

```sql
ALTER TABLE sampleTable MODIFY colName INTEGER NOT NULL UNIQUE;
```

---

## 테이블 생성과 동시에 제약 정의하기

```sql
CREATE TABLE sampleTable(
    no INTEGER NOT NULL,
    id INTEGER NOT NULL UNIQUE,
    CONSTRAINT pkey PRIMARY KEY(no)
);
```

---

## PrimaryKey 와 Unique 의 차이점
> 둘은 한개의 값만 가지는 유니크함에서는 같은것 같은데 무엇이 다른지 알아봅니다.

|Unique|primaryKey|
|------|-----|
|중복 방지용 | 중복 방지 + key 식별용|
|NULL 허용 O | NULL 허용 X|
|여러열에 지정가능|한 열에만 지정가능|

<br>

### 목적에 대한 차이가 있습니다.
* Unique와 primaryKey는 중복되는 값을 가지지 못하게 하는 공통점이 있으나, primary key는 말그대로 `key` 의 역할을 하는것입니다.
    * 이로 인해 여러 열에서 설정이 불가능합니다
    * `NULL`이 불가능합니다.

---

## Reference
* [SQL 첫걸음](http://www.yes24.com/Product/Goods/22744867)
* [QASTACK](https://qastack.kr/dba/15572/differences-between-unique-key-and-primary-key)