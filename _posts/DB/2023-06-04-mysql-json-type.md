---
title: MYSQL Json type 컬럼 사용법
date: 2023-06-04-18:30
categories:
- DB

tags:
- DB
- DataBase
- JPA
- MYSQL

---

## MySQL json type 사용법
> RDMBS 를 다루는 입장에서, 자연스럽진 않지만 예외상황이 필요한 경우를 위해 방법론 정도만 알아둡니다.

---

## Goal
- Mysql 에서 json type 을 사용하는방법을 정리해봅니다.
- spring + jpa 환경에서 사용하는 방법을 알아봅니다

---

## 조회

```sql
SELECT JSON_EXTRACT(json_column_name, '$.name')
   FROM table_name
```

- name 이라는 key 값에 따른 value 를 쿼리로 뽑고 싶은경우 위와같이 처리할 수 있습니다.


```sql
SELECT JSON_EXTRACT(json_column_name, "$.name"), JSON_EXTRACT(json_column_name, "$.age")
FROM table_name
WHERE JSON_EXTRACT(json_column_name, "$.age") > 1
ORDER BY JSON_EXTRACT(c, "$.name");
```

- where 절과 order by 조건을 주어 쿼리할 수 도 있습니다.

---

## 수정

### JSON_SET
> `JSON_SET` 을 이용해서 값을 변경해 주거나 추가해줄 수 있습니다.

```sql
update table_name
set json_column_name = JSON_SET(json_column_name, '$.name', 'abcd, '$.age', 13)
where id = 1;
```
- name 을 abcd, age 를 13으로 변경하고 싶은경우 위와 같은 쿼리로 처리할 수 있습니다.

- 이 밖에도 Insert, Replace 함수가 존재하는데 옵선들은 아래와 같습니다.
- [JSON_SET()](https://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#function_json-set) replaces existing values and adds nonexisting values.
- [JSON_INSERT()](https://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#function_json-insert) inserts values without replacing existing values.
- [JSON_REPLACE()](https://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#function_json-replace) replaces only existing values.

---

### JSON_INSERT 
> 조회, 수정을 이용한 응용

```sql
UPDATE table_name
SET json_column_name = JSON_INSERT(json_column_name, '$.NEW', JSON_EXTRACT(json_column_name, '$.OLD')),
    json_column_name = JSON_REMOVE(json_column_name, '$.OLD')
WHERE id = 1;
```

- `OLD` 라는 key 값을 가지고 있던 데이터를 위와같은 쿼리로 키값을 `NEW` 로 바꿀 수 있습니다.


### 디폴트값 설정해주기
> 5.8 이상버전

```sql
ALTER TABLE table_name ADD COLUMN json_column_name JSON NOT NULL DEFAULT ('{}') ;
ALTER TABLE table_name ADD COLUMN json_column_name JSON NOT NULL DEFAULT (JSON_OBJECT()) ;
```

- null 로 저장되는 경우, 디폴트값을 가지도록 `mysql 5.8` 이상버전부터는 설정해줄 수 있습니다.


> A JSON column cannot have a non-NULL default value.

- 하지만 [docs](https://dev.mysql.com/doc/refman/5.7/en/json.html) 에 따르면, 5.7 버전에서는 default 설정이 불가능 합니다.

```kotlin
ALTER TABLE table_name ADD json_column_name values JSON
UPDATE table_name SET json_column_name = JSON_OBJECT() WHERE 1 = 1
ALTER TABLE json_column_name JSON NOT NULL
```

- 만약 json 컬럼이 추가된뒤에 마이그레이션이 필요하다면, 위와같은 쿼리를 넣어서 기존 존재하는 컬럼들에게 `('{}')` 를 넣어줄 수 있습니다.

---

## Json 안에 Json 이 있는 형태의 경우

```sql
-- 조회
SELECT JSON_EXTRACT(json_column_name, '$inner_json.name')

-- 수정
UPDATE table_name
SET json_column_name = JSON_SET(json_column_name, '$inner_json.name', 'goodall'),
WHERE id = 1;
```

- `.` 을 이용해서 접근 및 수정도 가능합니다.

---

## JPA

### 의존성 추가

```gradle
implementation("com.vladmihalcea:hibernate-types-52:1.0.0")
```

### Map 을 이용해 json 형태로 저장하기

```kotlin
@TypeDef(name = "json", typeClass = JsonStringType::class)
@Entity
class JsonEntity(
    @Type(type = "json")
    @Column(name = "json_body", nullable = false, columnDefinition = "json")
    val jsonBody: Map<*, *>,

    @Column(name = "id", nullable = false)
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long = 0L
)
```

- `@TypeDef` 을 이용해서, `json` 이라는 이름을 가진 타입을 String 형태로 저장한다는것을 명시해줍니다.
- 이후, Map 으로 변수컬럼을 정해줍니다.

```kotlin
@Component
class InitLoader6(
    private val jsonEntityRepository: JsonEntityRepository,
) : ApplicationRunner {
    @Transactional
    override fun run(args: ApplicationArguments?) {
        val bodyMap = mapOf("name" to "yoonsung", "nick_name" to "goodall")
        jsonEntityRepository.save(
            JsonEntity(jsonBody = bodyMap)
        )
    }
}
```

- 실제로 디비에 저장된 형태를 보면, json 형태로 변경되어서 저장된것을 확인할 수 있습니다.


---

## Conclusion
- Mysql 에서 Json 타입의 컬럼을 사용할 수 있다.
- 주로 많이 쓰는 버전인 5.7 의 경우에는 5.8 미만이라서, 디폴트 값 설정이 불가능하다.

---

## Reference
- https://mvnrepository.com/artifact/com.vladmihalcea/hibernate-types-52/1.0.0
- https://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html
- https://dev.mysql.com/doc/refman/5.7/en/json.html
- https://stackoverflow.com/questions/61169030/mysql-set-default-value-to-a-json-type-column
- https://www.tutorialspoint.com/set-default-value-to-a-json-type-column-in-mysql