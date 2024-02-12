---
title: Index 로 인해 의도치 않은 정렬이 이루어질 수 있다.
date: 2024-02-12-18:00
categories:
- DB

tags:
- DB
- DataBase
- MYSQL
- TroubleShooting

---

## Index 를 이용해 데이터를 조회할때 자동 정렬되는 케이스를 알아봅니다.
> 조회할때 정렬조건을 명시하지 않는경우, Index 조회 컬럼으로 오름차순 정렬이 될수 있습니다.

---

## Goal
- Index 컬럼을 조회할때 인덱스 컬럼 조건으로 자동 정렬되는 경우를 알아봅니다.

---

## 복합 인덱스가 걸린 엔티티
> name, priority 순으로 설정된 복합인덱스

```kotlin
package com.example.kopring.member.domain

import com.example.kopring.BaseEntity
import java.time.ZonedDateTime
import javax.persistence.*
import javax.validation.constraints.NotBlank


@Table(name = "member", indexes = [Index(name = "idx__name_priority", columnList = "name,priority")])
@Entity
class Member(
    name: String,
    nickName: String = "123",

    createdAt: ZonedDateTime = ZonedDateTime.now(),

    @Column(name = "priority", nullable = false)
    val priority: Int = 0,

    @Column(name = "id", nullable = false)
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long = 0L
) : BaseEntity(createdAt = createdAt) {

    @Column(name = "name", nullable = false)
    var name = name
        protected set

    @Column(name = "nickName", nullable = false)
    var nickName = nickName
        protected set

    fun changeName(name: String) {
        this.name = name
    }
}
```

- Member Entity 에 `name, priority` 순으로 복합인덱스를 걸어주었습니다.
- 참고로 `indexes` 와 `columnList` 를 이용해 JPA 를 이용해 엔티티에 인덱스를 걸어주었습니다. (해당 방법에 대한 내용은 해당 포스팅에서 자세히 다루진 않습니다.)

### 데이터 저장

```kotlin
@Order(1)
@Component
class InitLoader(
    private val memberRepository: MemberRepository,
) : ApplicationRunner {
    @Transactional
    override fun run(args: ApplicationArguments?) {
        memberRepository.save(
            Member(name = "jys", priority = 3, nickName = "fortune")
        )

        memberRepository.save(
            Member(name = "jys", priority = 1, nickName = "goodall")
        )
    }
}
```

- member 는 같은 name (jys) 을 가진, 다른 prirority 값을 지닌 두개를 저장 해두겠습니다.
- entity 의 id 는 auto-increment 된 값을 사용하므로, priority=3 이 Id=1, priority=1 이 Id=2 순으로 데이터가 저장됩니다.

---

## 조회
> id 가 아닌 prirority 순으로 자동정렬이 진행됩니다.

```kotlin
@IntegrationTest
class MemberRepositoryTest(
    private val memberRepository: MemberRepository
) {
    @DisplayName("인덱스 오름차순으로 정렬된다.")
    @Test
    fun test1(){
        val result = memberRepository.findAllByName("jys")

        result[0].priority shouldBe 1
        result[1].priority shouldBe 3
    }
}
```

- 정렬조건을 주지 않은채 jys 라는 이름을 가진 데이터를 조회해보면, priority 순으로 정렬되어 데이터가 조회되는것을 볼 수 있습니다.
- 이렇게 자동 정렬이 되는 이유는, Mysql 의 경우 **기본적으로는** 인덱스를 B-Tree 를 통해서 관리하게 되고 이때문에 자동 정렬된 구조를 가지게 되어 조회시에 priority 순으로 정렬된 데이터 결과값을 얻게됩니다.

---

## [참고] Index 를 사용할때 사용하는 자료구조
> 복합적이고, 어떤 엔진을 사용하느냐에 따라 달라집니다.

> Most MySQL indexes (PRIMARY KEY, UNIQUE, INDEX, and FULLTEXT) are stored in [B-trees](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_b_tree). Exceptions: Indexes on spatial data types use R-trees; MEMORY tables also support [hash indexes](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_hash_index); InnoDB uses inverted lists for FULLTEXT indexes.

- [공식문서 내용](https://dev.mysql.com/doc/refman/8.0/en/mysql-indexes.html)
- 엔진별 사용 인덱스 [docs](https://dev.mysql.com/doc/refman/5.7/en/storage-engines.html)


### InnoDB 
> B-tree

![innodb](https://user-images.githubusercontent.com/43930419/158030984-851913b7-704f-4af5-8ab9-93e30220a9b7.png)


- B-tree 를 기본으로하고, [적응형 해시](https://dev.mysql.com/doc/refman/5.7/en/innodb-adaptive-hash.html)를 사용합니다.

### memory engine
> Hash Index + B-tree

![image](https://user-images.githubusercontent.com/43930419/158030931-87094e30-6e4e-42c4-b4f2-d813bbca172e.png)

---

## Conclusion
- 인덱스를 관리하는 자료구조인 B-Tree 영향으로 인해, 인덱스를 이용한 데이터 조회시 의도치 않은 정렬이 발생할 수 있다.
- 데이터 정렬을 의도하는경우, 명시적으로 쿼리에 정렬조건을 넣어주자.

## Code
- 관련 코드는 [Github](https://github.com/unluckyjung/kotlin-spring-jpa-playground/tree/db/index-sort) 에서 보실 수 있습니다.

---

## Reference
- https://dev.mysql.com/doc/refman/5.7/en/innodb-indexes.html
- https://dev.mysql.com/doc/refman/5.7/en/storage-engines.html
