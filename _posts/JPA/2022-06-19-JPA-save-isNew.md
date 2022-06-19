---
title: JPA save 동작 과정과 isNew()
date: 2022-06-19-18:00
categories:
- JPA

tags:
- JPA

---

## JPA의 save 동작 과정과 isNew()
> save는 **persist** or **merge** 로 동작합니다. 분기를 `isNew()` 로 확인합니다.

## Goal
- JPA에서 `save()` 메소드 동작 과정을 들여다 봅니다.
- **persist** 와 **merge** 분기조건이 되는 `isNew()` 의 조건에 대해서 알아봅니다.
- `isNew()` 의 로직을 엔티티에서 결정하는법을 알아봅니다.

---

## save 메소드 호출시, 동작하는 과정이 다를 수 있습니다.
> `merge()` 와 `persist()` 둘중 하나로 동작합니다.

> SimpleJpaRepository

![image](https://user-images.githubusercontent.com/43930419/172106914-f83cdb7f-90b7-4f7f-86e3-643991214a27.png)

- save의 동작과정을 들여다보면, `isNew()` 의 반환조건에 따라서, persist 와 merge 로 분기가 나뉘는것을 볼 수 있습니다.
- `isNew()`를 통해 Entity가 새롭게 만들어진 entity로 인지, 혹은 기존에 사용되던 Entity 인지를 구분합니다.
- `persist()` 의 경우 기존에 존재하던 entity로 인지하고, 일반적으로 바로 **insert 쿼리**가 나가게 됩니다.
- 하지만 `merge()` 의 경우, 밀어 넣으려는 값의 id가 테이블에 있는지를 있는지를 확인해보기 위해서 select 쿼리가 추가적으로 1회 나갈 가능성이 있습니다. (1차 캐시에 없는 경우)
  - 이 때문에 merge 사용시 `saveAll()` 과 같이 N개의 엔티티를 save 하게 되는경우, 불필요한 쿼리(select) N번이 추가적으로 발생하게 되어 성능에 이슈가 될 수 있습니다.
  - 또한 merge는 pk가 같은 기존에 entity를 대체해버리기 때문에, entity내의 필드값들이 의도치 않게 사라지거나 변경되는 사이드 이펙트가 발생할 수 있습니다.


---

## isNew()의 동작과정
> 해당 Entity가 새롭게 만들어진 Entity인지, 혹은 기존에 사용되던 Entity 인지를 구분합니다.


> AbstractEntityInformation.kt

![image](https://user-images.githubusercontent.com/43930419/172107016-9121037c-19f1-4934-85a0-f785cdae6405.png)

- `50-56` 라인을 보면 `isNew()` 판단 조건을 확인할 수 있습니다.
- **wapper type**: null 인지를 확인합니다.
- **primitive type**: 숫자 타입이면서, 값이 0인지를 확인합니다.


## isNew()를 고려해서 id를 정해줄때 고려할점
> uuid를 PK로 사용하는 경우

```kotlin
@Entity
class Member2(

    @Column(name = "member2_uuid")
    @Id
    val uuid: String = ""
) : BaseEntity()

...

val member = member2Repository.save(Member2("testUuid"))
```

- 위와 같이 Member Entity의 PK 를 String인 uuid로 잡는 경우 `isNew()`에서 false가 되어, save 호출시 새로운 엔티티임에도 불구하고 항상 **merge**를 호출하게 됩니다.
- 참고로 uuid를 pk로 잡는것은 정렬된 상태를 유지하는 [클러스티드 인덱스](https://unluckyjung.github.io/db/2022/05/08/clustered-index/)를 고려했을때, 권장하지는 않습니다. 

```kotlin
2022-06-06 16:36:56.856  INFO 10629 --- [           main] p6spy                                    : #1654501016856 | took 7ms | statement | connection 3| url jdbc:h2:mem:b32e5602-a5bf-4ce5-90cf-0325c3236a73
select member2x0_.member2_uuid as member1_1_0_, member2x0_.created_at as created_2_1_0_, member2x0_.updated_at as updated_3_1_0_ from member2 member2x0_ where member2x0_.member2_uuid=?
select member2x0_.member2_uuid as member1_1_0_, member2x0_.created_at as created_2_1_0_, member2x0_.updated_at as updated_3_1_0_ from member2 member2x0_ where member2x0_.member2_uuid='testUuid';
2022-06-06 16:36:56.907  INFO 10629 --- [           main] p6spy                                    : #1654501016907 | took 2ms | statement | connection 3| url jdbc:h2:mem:b32e5602-a5bf-4ce5-90cf-0325c3236a73
insert into member2 (created_at, updated_at, member2_uuid) values (?, ?, ?)
insert into member2 (created_at, updated_at, member2_uuid) values ('2022-06-06T16:36:56.870+0900', '2022-06-06T16:36:56.870+0900', 'testUuid');
2022-06-06 16:36:56.937  INFO 10629 --- [           main] p6spy                                    : #1654501016937 | took 0ms | rollback | connection 3| url jdbc:h2:mem:b32e5602-a5bf-4ce5-90cf-0325c3236a73
```

- 실제로 나가는 쿼리를 확인해보면, 불필요한 select 쿼리가 나가는것을 확인할 수 있습니다.

### Persistable 인터페이스를 구현해줍니다.
> `isNew()` 로직을 엔티티에서 정할 수 있습니다.

```kotlin
@MappedSuperclass
abstract class BaseEntity(
    @Column(name = "created_at")
    var createdAt: ZonedDateTime = DEFAULT_TIME,

    @Column(name = "updated_at")
    var updatedAt: ZonedDateTime = DEFAULT_TIME
) {
    ...
    companion object {
        val DEFAULT_TIME = ZonedDateTime.of(LocalDateTime.MAX, ZoneId.of("Asia/Seoul"))
    }
}
```

```kotlin
@Entity
class Member2(

    @Column(name = "member2_uuid")
    @Id
    val uuid: String = ""
) : Persistable<String>, BaseEntity() {

    override fun getId(): String {
        return uuid
    }

    // Persistable 인터페이스 구현 후 isNew() 로직 재설정
    override fun isNew(): Boolean {
        return super.createdAt == DEFAULT_TIME
    }
}
```

- 새로운 Entity 임을 `createdAt` 을 이용하여 구분하게 해줄 수 있습니다.
- **save** 가 되기전까지 `createdAt` 시간을 `DEFAULT_TIME` 으로 잡아줍니다.
- `Persistable<String>`인터페이스를 구현하여, `isNew()`의 판단 조건을 id가 아닌 createdAt 기준으로 판단하게 하여, `persist()`로 작동하게 할 수 있습니다.

---


## Conclusion
- `save()` 는 `isNew()` 를 통해서 새로운 객체인경우에는 persist, 아닌경우에는 merge 로 작동한다. merge 의 경우 불필요한 쿼리가 나가거나 의도하지 않은 값들을 교체해버리는 사이드 이펙트가 있을 수 있다.
- `isNew()` 는 primitive + Number type 인경우 **0**, unprimitive 인 경우에는 **null** 인지를 확인해서 새로운 객체인지를 확인한다.
- Persistable 인터페이스를 구현하여 `isNew()` 판단 조건을 정해줄 수 있다.


---

## Reference
- https://stackoverflow.com/questions/49747891/kotlin-jpa-entity-id
- https://leegicheol.github.io/jpa/jpa-is-new/
- https://insanelysimple.tistory.com/m/311