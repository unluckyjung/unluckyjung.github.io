---
title: JPA Cascade 와 OrphanRemoval
date: 2022-11-29-18:00
categories:
- JPA

tags:
- JPA

---

## JPA Cascade & OrphanRemoval
> `Cascade` 는 엔티티간의 생명주기를 같이 다룰 수 있습니다. `OrphanRemoval` 는 객체상의 변화가 DB 까지 적용됩니다.

---

## Goal
- `Cascade` 와 `OrphanRemoval` 옵션에 대해서 알아봅니다.
- 각 옵션을 사용할때 주의할점을 알아봅니다.

---

## Entity Setting
> Team(상위 / 부모) MyMeber(하위 / 자식)

### 자식엔티티
> `MyMember.kt`

```kotlin
@Where(clause = "deleted_at is null")
@SQLDelete(sql = "UPDATE my_member SET deleted_at = NOW() WHERE member_id = ?")
@Table(name = "my_member")
@Entity
class MyMember(
    val name: String,

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "team_id")
    var team: Team? = null,

    @Column(name = "deleted_at")
    var deletedAt: ZonedDateTime? = null,

    @Column(name = "member_id")
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long = 0L
)
```

### 부모 엔티티
> `Team.kt`

```kotlin
@SQLDelete(sql = "UPDATE team SET deleted_at = NOW() WHERE team_id = ?")
@Where(clause = "deleted_at is null")
@Table(name = "team")
@Entity
class Team(
    @Column(name = "name")
    val name: String,

    // 연관관계 주인은 MyMember(fk team_id 소유)
    @OneToMany(mappedBy = "team")   // 해당 옵션이 계속 바뀔예정
    val members: MutableList<MyMember> = mutableListOf(),

    @Column(name = "deleted_at")
    var deletedAt: ZonedDateTime? = null,

    @Column(name = "team_id")
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long = 0L
) {
    fun addMember(member: MyMember) {
        members.add(member)
        member.team = this
    }

    fun addAllMember(members: List<MyMember>) {
        members.forEach {
            addMember(it)
        }
    }
}
```

- Team(One) MyMember(Many) 양방향 매핑을 맺어주었습니다.

---

## Cascade

### Cascade 비활성화
> Team 에 Member 를 넣어두고, `TeamReposity` 만을 이용해 저장하는 경우

```kotlin
    // 연관관계 주인은 MyMember(fk team_id 소유)
    @OneToMany(mappedBy = "team")   // Cascacde 옵션을 주지 않음.
    val members: MutableList<MyMember> = mutableListOf(),
```

```kotlin
@Order(1)
@Component
class InitLoader(
    private val teamRepository: TeamRepository,
) : ApplicationRunner {

    @Transactional
    override fun run(args: ApplicationArguments?) {
        Team("team1").apply {
            this.addMember(MyMember("member1"))
            this.addMember(MyMember("member2"))
        }.let(teamRepository::save)

        Team("team2").apply {
            this.addAllMember(listOf(MyMember("member3"), MyMember("member4")))
        }.let(teamRepository::save)
    }
}
```


![image](https://user-images.githubusercontent.com/43930419/204083023-bd08662f-f673-4426-9500-e23d2f2dbd3a.png)

![image](https://user-images.githubusercontent.com/43930419/204083017-c5cc6a7f-8278-40a9-b4ed-dfe6ae41eba1.png)


- 양방향 매핑이여도 Team 이 주체이기 때문에, **Cascade** 옵션이 **NONE** 이면 Team 만 저장되고 **Member 는 같이 DB에 저장되지 않습니다.**


### CascadeType.PERSIST

```kotlin
    // 연관관계 주인은 MyMember(fk team_id 소유)
    @OneToMany(mappedBy = "team", cascade = [CascadeType.PERSIST])
    val members: MutableList<MyMember> = mutableListOf(),
```

![image](https://user-images.githubusercontent.com/43930419/204083118-901e17c3-1826-40f8-b5ff-568365555828.png)

- Team과 함께 Member도 같이 저장이 되는것을 확인 할 수 있습니다.

> PERSIST 옵션에서 Team 을 `TeamRepository` 을 이용해 삭제시

```kotlin
@Order(2)
@Component
class InitLoader2(
    private val teamRepository: TeamRepository,
) : ApplicationRunner {
    @Transactional
    override fun run(args: ApplicationArguments?) {
        teamRepository.deleteById(1L)
    }
}
```

![image](https://user-images.githubusercontent.com/43930419/204083184-99b2406f-3b51-4e91-8aff-83dddf4df324.png)

![image](https://user-images.githubusercontent.com/43930419/204083211-fd9d0a89-d9dc-4e5c-bc8b-addcce8c90a1.png)

- Team 객체는 삭제된것이 보이나. Team 에 연관관계를 맺고 있던 Member 들은 DB에서 삭제되지 않는것을 확인할 수 있습니다.

### Cascade.ALL
> PERSIST + REMOVE

```kotlin
    @OneToMany(mappedBy = "team", cascade = [CascadeType.ALL])
    val members: MutableList<MyMember> = mutableListOf(),
```

![image](https://user-images.githubusercontent.com/43930419/204083397-382cf350-9548-45f0-8026-f1026b63efec.png)

- Team 을 삭제시 연관관계를 맺고 있던, Member 도 같이 삭제되는것을 확인할 수 있습니다. (생명주기를 같이하게 됩니다.)

### SoftDelete 적용시 주의
> 부모(Team) 쪽에서 반드시 SoftDelete 를 같이 구현해주어야 합니다.

```kotlin

// Team 쪽에도 반드시 SoftDelete 구현 해주어야함.
// 상위 엔티티(team) 에 softDelete 가 구현이 안된 경우 문제발생
//@SQLDelete(sql = "UPDATE team SET deleted_at = NOW() WHERE team_id = ?") 
@Where(clause = "deleted_at is null")
@Table(name = "team")
@Entity
class Team()

@Where(clause = "deleted_at is null")
@SQLDelete(sql = "UPDATE my_member SET deleted_at = NOW() WHERE member_id = ?")
@Table(name = "my_member")
@Entity
class MyMember()
```

```kotlin
Caused by: org.springframework.dao.DataIntegrityViolationException: could not execute statement; SQL [n/a]; constraint ["FKSWUHMFB1E4KO9R2U5H51GCYJ: PUBLIC.MY_MEMBER FOREIGN KEY(TEAM_ID) REFERENCES PUBLIC.TEAM(TEAM_ID) (1)"; SQL statement:
```

<img width="1214" alt="image" src="https://user-images.githubusercontent.com/43930419/204096146-9bed8197-1633-4591-b883-32c23a15d5a4.png">

<img width="1603" alt="image" src="https://user-images.githubusercontent.com/43930419/204096046-d4078bf9-e0f4-4a0d-abda-de6e91890af1.png">

- MyMember 에만 `softDelete` 를 구현해둔 경우, Team 쪽에서 Cascade 로 삭제 시도시, HardDelete 로 동작해서, FK 참조 에러가 발생합니다. `JPA SQL Error: 23503`

```kotlin
1. 하위 엔티티 삭제 (N개 / SoftDelete)
2. 상위 엔티티 삭제 (HardDelet) -- 이 과정에서 에러 발생
```

- Cascade 옵션은 하위 엔티티를 먼저 삭제하고 상위 엔티티를 삭제 하는 순서로 동작하게 됩니다.
- 하위 엔티티만 SoftDelete 인 경우, 상위 엔티티를 삭제하는 과정에서 하위 엔티티와의 FK 관계가 아직 끊어지지 않은것으로 판단되어 FK 관련 에러가 발생하게 됩니다.


### 참고
> **상위 엔티티(Team)** 에만 `SoftDelete` 가 구현되고 상위엔티티를 삭제하는경우에는, 상위 엔티티는 `SoftDelete`, 하위 엔티티는 `HardDelete` 로 에러 없이 수행이 되어집니다.

---

## Delete 함수 직접 구현시 주의

```kotlin
@Entity
class Team(){
    fun delete() {
        deletedAt = ZonedDateTime.now()
    }
}
```

```kotlin
    override fun run(args: ApplicationArguments?) {
        val team = teamRepository.findByIdOrNull(1L) ?: throw IllegalStateException()
        // teamRepository.delete(team)
        team.delete()
    }
```


![image](https://user-images.githubusercontent.com/43930419/204097381-19e40cb0-1d5d-4e21-9003-9cda95f5fb0c.png)

![image](https://user-images.githubusercontent.com/43930419/204097370-daf9cd79-99d6-4cec-86b2-08278e1f73bb.png)


- softDelete 를 위해서 직접 delete 함수를 구현해주는 경우도 있습니다.
- Repository 가 아닌 해당 함수를 통해 삭제하는경우에는 `cascade.REMOVE` 옵션이 작동하지 않아, 부모 객체(Team) 만 삭제되게 됩니다.

```kotlin
@Entity
class Team(){
    fun delete() {
        // cascade.REMOVE 옵션이 안먹는다.
        val now = ZonedDateTime.now()
        deletedAt = now

        // 직접 값을 넣어주어야 한다.
        // CascadeType.PERSIST 때문에 값 변경이 적용 된다. member 가 직접 변경되는것이라 적용되어짐.
        members.forEach {
            it.deletedAt = now
        }
    }
}
```

- 이부분을 해결하기 위해서는 직접 하위 엔티티의 softDelete 컬럼값을 변경해 주어야 합니다. (MyMember에 `delete()` 함수를 구현해주고 호출하는것이 좋으나, 직관성을 위해 코드를 위와같이 구현했습니다.)


---

## OrphanRemoval

### Cascade 의 한계
> team 을 통해서 member 를 객체 관계(members)에서 끊어내는 경우 DB 에는 적용안됌

```kotlin
@Order(2)
@Component
class InitLoader2(
    private val teamRepository: TeamRepository,
) : ApplicationRunner {
    @Transactional
    override fun run(args: ApplicationArguments?) {
        val team = teamRepository.findByIdOrNull(1L) ?: throw IllegalStateException()
        team.members.removeAt(0)
    }
}
```

- 객체간의 연관관계는 끊어졌으나 `MyMember` 에 대해서 delete 쿼리가 나가지 않고, DB 상에서는 여전히 MyMember와 Team 둘다 그대로 남아있게 됩니다.
- 즉 List 에서는 삭제되었으나, **DB 상에서는 삭제가 되지 않습니다.**


### orphanRemoval = true

```kotlin
    @OneToMany(mappedBy = "team", cascade = [CascadeType.ALL], orphanRemoval = true)
    val members: MutableList<MyMember> = mutableListOf(),
```

![image](https://user-images.githubusercontent.com/43930419/204096719-e0b172ac-7056-43c7-ab63-ce955b7cc020.png)

- `orphanRemoval = true` 옵션을 활성화 해주는 경우, 부모 엔티티에서 자식 엔티티와의 관계를 객체상으로 끊어낼때, delete 쿼리가 발생하여 DB 상에서도 자식 엔티티가 삭제되는것을 확인할 수 있습니다.

### 주의할점

```kotlin
    @OneToMany(mappedBy = "team", cascade = [CascadeType.ALL], orphanRemoval = false) // orphanRemoval 비활성화
    val members: MutableList<MyMember> = mutableListOf(), 
```

```kotlin
class InitLoader2(
    private val teamRepository: TeamRepository,
) : ApplicationRunner {
    @Transactional
    override fun run(args: ApplicationArguments?) {
        val team = teamRepository.findByIdOrNull(1L) ?: throw IllegalStateException()
        team.members.removeAt(0)
        teamRepository.delete(team) // 문제 발생
    }
}
```

- `orphanRemoval = false` 인 경우 객체상에서 관계를 끊고, 부모 엔티티를 삭제하는 경우 `[SQL Error: 23503]` 와 함께 FK 에러가 납니다.

```
부모 객체(자식1, 자식2)

# 부모객체 가 들고있는 자식2 를 객체레벨에서 삭제
부모객체(자식1) -- 객체상 자식1개만 남아있음.

# 부모객체 삭제
1. 자식 1에 대해서 삭제쿼리 발생
2. 부모에 대해서 삭제 쿼리발생 -> DB 상에 자식2 가 남이있어서 에러발생
```

- 초기 부모 객체는 자식 객체를 2개가지고 있는 상황에서 1개의 자식객체와 연관관계를 끊고, `CASCADE REMOVE` 옵션을 통해 부모 삭제를 시도하면 보유하고 있는 자식 객체는 1개(자식2) 밖에 없다고 판단하게 됩니다. 
- 자식 엔티티에 대해 1개의 DELETE 쿼리만 발생하게 되어. DB 상에 여전히 남아있는 고아 객체(자식2) 로 인해 FK 참조에러가 발생하게 됩니다.

---

## Conclusion
- `CASCADE` 는 부모와 자식간의 생명주기를 같이하게 하는 옵션이다. (부모가 삭제되는경우 자식도 DB 삭제, 부모와 자식간의 연관관계를 맺고 부모를 저장하는 경우 자식도 같이 DB저장)
- `orphanRemoval` 의 경우, 부모와 자식간의 연관관계를 DB레벨에서 같이 관리하는 옵션이다. (부모에서 자식간의 연관관계를 끊으면 자식은 고아가 되는것이 아니라 DB 상에서도 삭제)
- 상황에 맞춰서 개발을 진행하되, 일반적으로는  `CASCADE = ALL`, `orphanRemoval = true` 로 가져가는것을 추천한다.