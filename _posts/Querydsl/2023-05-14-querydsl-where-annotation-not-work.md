---
title: Querydsl 사용시 SoftDelete 된 엔티티도 조회되는 문제 
date: 2023-05-14-13:00
categories:
- Querydsl

tags:
- Querydsl
- Spring
- Kotlin
- JPA

---

## Querydsl 사용시 SoftdDelete 된 엔티티가 조회가 되어버리는 이슈를 확인해봅니다.
> join 에 들어가는 엔티티에 대해서는 `@Where` 어노테이션이 동작하지 않는것을 알아봅니다.

---

## Goal
- Querydsl 사용시 `@Where` 조건이 동작하지 않는 이슈를 확인해봅니다.
- 해결방법을 알아봅니다.

---

## 엔티티 조건
> 두개의 엔티티 전부 `@Where(clause = "deleted_at is null")`  어노테이션을 달아주어 SoftDelete 된 조건은 조회해오지 않게 하였습니다.

### Member

```kotlin
@SQLDelete(sql = "UPDATE member SET deleted_at = NOW() WHERE member_id = ?")
@Where(clause = "deleted_at is null")
@Table(name = "member")
@Entity
class Member(
    @Column(name = "member_name", nullable = true)
    val name: String? = null,

    @Column(name = "member_age", nullable = false)
    val age: Int = 0,

    team: Team? = null,

    deletedAt: ZonedDateTime? = null,

    @Column(name = "member_id")
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long = 0L,
) {

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "team_id")
    var team = team
        protected set

    fun changeTeam(team: Team) {
        this.team = team
        team.members.add(this)
    }

    @Column(name = "deleted_at")
    var deletedAt: ZonedDateTime? = deletedAt
        protected set

    fun delete(deletedAt: ZonedDateTime = ZonedDateTime.now()) {
        this.deletedAt = deletedAt
    }
}
```


### team

```kotlin
@SQLDelete(sql = "UPDATE team SET deleted_at = NOW() WHERE team_id = ?")
@Where(clause = "deleted_at is null")
@Table(name = "team")
@Entity
class Team(

    @Column(name = "team_name", nullable = false)
    val name: String,

    @OneToMany(mappedBy = "team", orphanRemoval = true, cascade = [CascadeType.ALL])
    val members: MutableList<Member> = mutableListOf(),

    deletedAt: ZonedDateTime? = null,

    @Column(name = "team_id")
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long = 0L,
) {
    override fun toString(): String {
        return "Team(name='$name', id=$id)"
    }

    @Column(name = "deleted_at")
    var deletedAt: ZonedDateTime? = deletedAt
        protected set

    fun delete() {
        val now = ZonedDateTime.now()
        deletedAt = now

        members.forEach {
            it.delete()
        }
    }
}
```

---

## from 절에 있는 엔티티 조회시
> 엔티티의 `@where` 어노테이션에 걸려있는 쿼리가 추가되어 나갑니다.

### Dao

```kotlin
@Repository
class TeamDao : QuerydslRepositorySupport(Team::class.java) {

    @Transactional(readOnly = true)
    fun findTeamById(teamId: Long): Team? {
        return from(team)
            .where(team.id.eq(teamId))
            .select(team)
            .fetchOne()
    }
}

@Repository
class MemberDao : QuerydslRepositorySupport(Member::class.java) {

    @Transactional(readOnly = true)
    fun findMemberById(memberId: Long): Member? {
        return from(QMember.member)
            .select(QMember.member)
            .where(QMember.member.id.eq(memberId))
            .fetchOne()
    }
}
```

### 조회 결과

```kotlin
@DisplayName("softDelete 된 Team 이 from 으로 시작되면 조회되지 않는다.")
@Test
fun softDeleteTest1() {
    val team = teamRepository.save(
        Team(name = "team1", deletedAt = ZonedDateTime.now()),
    )

    teamDao.findTeamById(team.id) shouldBe null
}

@DisplayName("softDelete 된 member 가 from 으로 시작하면 조회되지 않는다.")
@Test
fun softDeleteTest2() {
    val member = memberRepository.save(
        Member(name = "member1", deletedAt = ZonedDateTime.now()),
    )

    memberDao.findMemberById(member.id) shouldBe null
}
```


![image](https://user-images.githubusercontent.com/43930419/238194238-2b59c1ec-5f18-4851-b01a-27864bf9a313.png)

- 쿼리를 확인해보면 `@Where(clause = "deleted_at is null")` 조건에 따라서 필터링이 되는것을 확인할 수 있습니다.

---

## join 에 들어가는 엔티티는 @where 어노테이션이 동작하지 않습니다.
> SoftDelete 된 엔티티가 조회될 수 있습니다.

### Dao

```kotlin
@Transactional(readOnly = true)
fun findMembersByTeam(teamId: Long): List<Member> {
    return from(team)
        .join(member).on(member.team.eq(team))
        .where(team.id.eq(teamId))
        .select(member)
        .fetch()
}
```

### 조회 결과

```kotlin
@DisplayName("softDelete 된 member 가 join 조건절에 있으면 where 쿼리가 나가지 않아 무시하고 조회되어버린다.")
@Test
fun softDeleteTest3() {
    val team = teamRepository.save(
        Team(name = "team1"),
    )
    val softDeletedMember = memberRepository.save(
        Member(name = "member1", team = team, deletedAt = ZonedDateTime.now()),
    )

    val result = teamDao.findMembersByTeam(teamId = team.id)

    result.size shouldBe 1
    result[0] shouldBe softDeletedMember
    result[0].team shouldBe team
}
```

![image](https://user-images.githubusercontent.com/43930419/238194241-e3862fe6-1464-4d51-9c56-726227184780.png)


- team 에 대해서만 `@where` 어노테이션의 조건이 추가되고, member 엔티티에  `@where` 는 동작하지 않아서 softDelete 된 member 엔티티도 조회되어버립니다.


---

## 해결방법
> 조건을 쿼리에 명시적으로 추가 해줍니다.

```kotlin
@Transactional(readOnly = true)
fun findMembersByTeam2(teamId: Long): List<Member> {
    return from(team)
        .join(member).on(member.team.eq(team), member.deletedAt.isNull) // isNull 추가
        .where(team.id.eq(teamId))
        .select(member)
        .fetch()
}
```

![image](https://user-images.githubusercontent.com/43930419/238194246-26708e08-ae2e-4eef-8b60-0cb630af88a6.png)

- softDelete 조건으로 따지는 `delted_at` 에 isNull 을 추가해주면 조건이 추가되어 걸러지는것을 확인할 수 있습니다. 


---

## Conclusion
- QueryDsl 사용시 join 에 들어가는 엔티티 테이블에 대해서는 `@Where` 어노테이션이 동작하지않아, 원하지 않는 결과가 나올수 있다.
- join 에 들어가는 엔티티에 대해서는 미조회 조건을 쿼리에 넣어주자.

---

## Code
- 관련된 코드는 [Github](https://github.com/unluckyjung/kotlin-querydsl-practice) 에서 볼 수 있습니다.