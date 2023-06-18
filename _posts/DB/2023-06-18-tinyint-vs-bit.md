---
title: TINYINT 와 BIT 차이 그리고 BOOLEAN
date: 2023-06-18-13:30
categories:
- DB

tags:
- DB
- DataBase
- MYSQL

---

## TINYINT 와 BIT 의 차이점을 알아봅니다.
> BOOLEAN 타입을 표현하고 싶을때는 TINYINT 를 사용하는것이 일반적입니다. (MYSQL 기준)

---

## Goal
- `TINYINT` 와 `BIT` 의 차이점을 알아봅니다.
- True/false 를 표현하는 **BOOLEAN** 타입을 나타내고 싶을때, 둘중 어떤것을 쓰는게 좋을지 고민해봅니다.
- **MYSQL** 에선 어떻게 처리하고 있는지 확인해봅니다.

---

## 공통점
- TINYINT와 BIT 타입은 둘 다 작은 크기의 데이터를 저장할 때 사용되는 데이터 타입 입니다.
- 즉 BOOLEAN 을 표현하기위해 존재하는 타입이 아니고, **BOOLEAN 을 나타낼 수 있는 방식의 타입 입니다.**

---

## 차이점

### 저장 방식
- **TINYINT**
  - 8-bit, 
  - signed/unsigned integer 데이터 타입입니다. 
  - 즉 **정수값**이고 1byte(8 bit) 의 고정 크기를 가집니다.
- **BIT**
  - 1-bit, binary 데이터 타입입니다. 
  - 1 byte보다 작은 크기를 가지며, 0 또는 1의 값을 저장할 수 있습니다.


### 표현 방식
- **TINYINT**
  - 0 또는 1의 값을 저장하는 경우, TINYINT 타입으로 정의하면 됩니다. 
  - 이 때, 0은 false, 1은 true 로 해석할 수 있습니다.
- **BIT**
  - BIT 타입으로 정의하면, 0 또는 1의 값을 가진 이진수로 저장됩니다. 
  - `0b0` 또는 `0b1`로 저장되며, 이를 해석하여 boolean 값으로 사용할 수 있습니다.


---

## Boolean 타입 표현시 어떤것을 사용하는게 좋을까?
> MYSQL 기준에서는 **TINYINT** 사용 권장

- MySQL에서 boolean 필드를 선언할 때 **TINYINT 타입을 사용하는 것이 좋다고 합니다.**
- MySQL에서는 BIT 타입으로 컬럼을 선언해도 결국 byte 단위로 데이터를 저장하기 때문에 저장 공간의 잇점을 얻기도 불가능하여, boolean 값을 표현하고 싶을때는 TINYINT 타입이 더 적합하다고 합니다. (os 에서 page 크기가 Fix 인것과 비슷한 맥락으로 이렇게 설계되어있는것으로 추측)


![image](https://github.com/unluckyjung/unluckyjung.github.io/assets/43930419/a0309275-96cd-4f74-86cb-5a0aafc81880)


- 또한 `mysql 5.7` 기준에서는 BOOLEAN 을 쓰면 자동으로 TINYINT 로 처리되고 있습니다. [docs](https://dev.mysql.com/doc/refman/5.7/en/other-vendor-data-types.html)

> 필자의 경우에는 위와 같은 이유때문에 MYSQL 를 사용하는 상황에서 BOOLEAN 을 표현하고 싶은 경우에는 명시적인 이유로 `BOOLEAN` 혹은 `BOOL` 로 타입을 선언해 사용하고 있습니다.

## 참고
> Since there's no semantically meaningful difference between the two data-types when it comes to representing Boolean values, neither data-type is "more correct". But, there are downsides to using a binary field in order to represent a true-false value:

> The [MySQL driver for Node.js returns BIT data as a Buffer](https://www.bennadel.com/blog/3188-casting-bit-fields-to-booleans-using-the-node-js-mysql-driver.htm); because, of course it would. After all, a BIT column represents binary data, which is what the Buffer represents in Node.js. As such, in order to translate a BIT(1) result into a true / false data-type, you have add special type-casing logic to your database client configuration.

> The [MySQL driver for Java returns binary data when BIT is used in a COALESCE() call](https://www.bennadel.com/blog/2871-using-bit-values-in-coalesce-in-mysql-results-in-binary-values.htm). As such, if you are performing a LEFT OUTER JOIN on a BIT field and attempt to provide a default value with COALESCE(), you have to CAST() the resultant value back to an UNSIGNED type in order to use the value as a Truthy within your application code.

- 오피셜한 정보는 아니지만, java, node js 환경에서 Mysql 드라이버의 경우, bit 타입을 사용한다면 추가적인 형변환 로직을 타야해서 더 느리다는 이야기도 있습니다.

---

## Conclusion
- TINYINT, BIT 둘다 **Boolean** 을 표현하고 싶을때 사용할 수 있는 타입이나, 저장, 표현방식은 다르다.
- MYSQL 5.7 기준에서는 `TINYINT` 이 권장되고, `BOOLEAN` 이나, `BOOL` 으로 타입을 지정해줄 수 있다. (내부적으로는 `TINYINT` 사용)

---

## Reference
- https://dev.mysql.com/doc/refman/5.7/en/other-vendor-data-types.html
- https://www.bennadel.com/blog/3845-why-i-use-tinyint-columns-instead-of-bit-columns-for-boolean-data-in-a-mysql-application.htm