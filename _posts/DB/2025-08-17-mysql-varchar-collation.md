---
title: DB column varchar 속성의 경우 대소문자 구분시 인코딩 방식 주의하기
date: 2025-08-17-14:50
categories:
- DB

tags:
- DB
- DataBase
- MYSQL
- TroubleShooting

---

## MySQL의 VARCHAR 컬럼은 Collation 설정에 따라 대소문자를 구분하거나 구분하지 않는다.
> 의도치 않은 중복 저장 또는 조회 누락을 방지하려면 적절한 Collation 설정이 필요하다.

---

## Goal
- MySQL의 VARCHAR 컬럼 설정시 주의할 인코딩 방식에 대해서 알아봅니다.

---

## 선결론
- MySQL의 `VARCHAR` 컬럼은 `Collation` 설정에 따라 **대소문자를 구분할 수도 있고**, **구분하지 않을 수도** 있다.
- 의도한 동작을 위해선 단순히 `CHARSET`만이 아니라, `COLLATE`까지 명시적으로 설정해야 한다.

---

## 상세내용
- 규칙 (MySQL 공통):
  - `_ci`: case-insensitive (대소문자 구분 없음)
  - `_cs`: case-sensitive (대소문자 구분)
  - `_bin`: binary 비교 (바이트 단위 비교)

- **보통 디폴트값은 `_ci`이다.**
  (예: MySQL 8.0의 기본값은 `utf8mb4_0900_ai_ci`) 인점에 주의 해야한다. [DOCS](https://dev.mysql.com/doc/refman/8.0/en/charset-server.html)

- `_ci`일 때:
  - 저장된 값이 `'ABCD'`일 경우 `'abcd'`를 추가하면 **중복 저장 에러 발생**
  - `'aBCD'`로 조회해도 `'ABCD'`가 **검색됨**

```sql
-- 현재 컬럼의 Collation 확인
SHOW FULL COLUMNS FROM `your_table`;

-- 컬럼을 대소문자 구분(collation 변경)으로 수정
ALTER TABLE your_table
  MODIFY your_column VARCHAR(36)
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_0900_as_cs;
```

---

## 추가 정보: `_cs` vs `_bin` 차이 및 실무 설정 팁

| 항목         | `_cs` (Case Sensitive) | `_bin` (Binary) |
|--------------|------------------------|------------------|
| 비교 방식     | 문자 단위 비교          | 바이트 단위 비교 |
| 예시         | `a` ≠ `A`, `ü` = `ü`   | `a` ≠ `A`, `ü` ≠ `ü` |
| 정렬         | 언어 규칙 기반 정렬     | ASCII 기준 정렬 |
| 주 용도      | 대소문자만 구분 필요할 때 | 완전한 바이트 일치 필요할 때 |
| 장단점       | 사람이 보기 쉬운 정렬     | 정렬 비직관적이지만 성능 우수 |

### 실무 추천:
- **기본 조회용 필드** → `_ci`
- **로그인 ID, 토큰 등 정확한 매칭 필드** → `_cs` 또는 `_bin`

```sql
-- 테이블 생성 시 COLLATE 설정을 통해 대소문자 구분을 명시한다.
CREATE TABLE users (
  id INT PRIMARY KEY AUTO_INCREMENT,
  username VARCHAR(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_as_cs, -- username 에는 cs 로 세팅한다.
  email VARCHAR(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci
);
```

---

## Conclusion
- `VARCHAR` 컬럼에서 대소문자 구분을 명확히 하고 싶다면 반드시 `COLLATE` 속성을 의도에 맞게 설정해야 한다.
- 특히 로그인 ID나 고유값 비교, 해시값 컬럼 등은 기본값인 `_ci`로 설정할 경우, **예상치 못한 중복 허용 또는 조회 결과 오류**가 발생할 수 있다.
- 실무에서는 **기본값은 `_ci`**, **정밀 비교에는 `_cs` 또는 `_bin`** 전략을 추천한다.

---

## Reference
- [MySQL 8.0 Character Set and Collation](https://dev.mysql.com/doc/refman/8.0/en/charset-server.html)