---
title: JPA OneToMany 단방향 연관관계 사용시 주의할점
date: 2023-04-20-18:50
categories:
- JPA

tags:
- JPA
- Kotlin

---

## OneToMany 단방향 연관관계시 쿼리 추가발생 문제를 정리해봅니다.
> null 을 채우는 쿼리가 추가적으로 나갈 수 있습니다.

---


## Goal
- OneToMany 단방향 연관관계시 쿼리 추가발생 문제를 정리해봅니다.

---

## Summary
- fk 관리문제 (DB상에는 Many 쪽에 fk가 존재, 하지만 엔티티 레벨에선 보이지 않음)
- 중간테이블이 자동적으로 생기는 문제 (이 때문에 `@JoinColumn` 통해서 컬럼 명시. 해당부분은 이글에서 언급하지 않음) 
- **연관관계 관리를 위해서 update 쿼리가 추가적으로 나가는 문제**
- **예상치 못한 null insert 및 update query 발생문제**
- 해당 글에서는 **bold** 처리한 내용 위주로 다룹니다.

---

## 엔티티 구조
> Team(One) Member(Many) , Team 에서만 Member 를 알고 있는 `OneToMany` **단방향** 연관관계

```kotlin
@SQLDelete(sql = "UPDATE team SET deleted_at = NOW() WHERE team_id = ?")
@Where(clause = "deleted_at is null")
@Table(name = "team")
@Entity
class Team(
    @Column(name = "name")
    val name: String,

    // OneToMany 단방향
    @JoinColumn(name = "team_id")
    @OneToMany(cascade = [CascadeType.ALL], orphanRemoval = true)
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
    }

    fun addAllMember(members: List<MyMember>) {
        members.forEach {
            addMember(it)
        }
    }
}

@Where(clause = "deleted_at is null")
@SQLDelete(sql = "UPDATE my_member SET deleted_at = NOW() WHERE member_id = ?")
@Table(name = "my_member")
@Entity
class MyMember(
    val name: String,

//    @ManyToOne(fetch = FetchType.LAZY)
//    @JoinColumn(name = "team_id")
//    var team: Team,
```

- Team 만 MyMember 를 알고있는 단방향 `OneToMany` 관계입니다.

---

## 저장시 발생하는 문제
> member가 들고있는 fk 를 null 로 채운뒤, fk를 설정하는 update 쿼리가 추가적으로 나가게 됩니다.

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
            this.addAllMember(listOf(MyMember("member3"), MyMember("member4", )))
        }.let(teamRepository::save)
    }
}
```

Team 을 통해서 Member와 연관관계를 맺고 저장시 순서는 아래와 같습니다.

1. 팀 엔티티 저장 
2. 멤버 엔티티 저장 (`team_id = null`)
3. 멤버 엔티티 업데이트 (`team_id = 1L` 1번에서 만든 team의 id 로 매핑)


<img width="674" alt="image" src="https://user-images.githubusercontent.com/43930419/232766971-d384ba2d-6870-469f-97fe-ac4da500a401.png">

- 멤버 엔티티의 팀 정보를 업데이트 하는 쿼리가, 멤버 엔티티 개수만큼 추가적으로 더 나가는것을 확인할 수 있습니다. 
- 팀에 즉 3명의 멤버가 있었다면, 3개의 fk 업데이트 쿼리가 추가적으로 발생하게 됩니다.

- 해당 이슈가 발생하는 원인은 `fk(team_id)` 가 Team 이 아닌 Member 쪽에 있기 때문에, JPA 레벨에서 나중에서야 fk 를 찾을 수 있기 때문입니다.

---

## 삭제시 발생하는 문제
> fk를 null 로 만드는 쿼리가 먼저 나가게 됩니다.

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
            this.addAllMember(listOf(MyMember("member3"), MyMember("member4", )))
        }.let(teamRepository::save)
    }
}

// 추가된 로직
@Order(2)
@Component
class InitLoader2(
    private val teamRepository: TeamRepository,
) : ApplicationRunner {
    @Transactional
    override fun run(args: ApplicationArguments?) {
        val team = teamRepository.findAll().firstOrNull()
        // team 을 삭제 -> cascade 옵션으로 MyMember 도 같이 삭제
        teamRepository.delete(team!!)
    }
}
```

<img width="517" alt="image" src="https://user-images.githubusercontent.com/43930419/232767477-f0c84592-7b8b-4720-bd8d-0b8be222a6a4.png">

![image](https://user-images.githubusercontent.com/43930419/232767656-b06c7cba-4e87-4249-9ba2-45c13b088259.png)

- MyMember 쪽 fk 를 null 로 채우는 쿼리가 추가적으로 나갑니다.
- 이 때문에, softDelete 후 에도 삭제전 fk 를 확인할 수 없어, 삭제전 Member가 어떤 team 과 관계가 맺어져있었는지 히스토리 파악이 불가능해집니다.


> fk 가 MyMember 쪽에 있어서 발생하는 이슈 입니다. (Child가 Parent 를 알지 못하는 구조)

1. Spring Data JPA가 일단 삭제되는 team 에 대해서 연관관계를 끊겠다고, 자체적으로 team=id 를 null로 변경하는 update 쿼리를 보냅니다.
2. 자식엔티티(MyMember) 에 Delete 쿼리가 나가게 됩니다. (`@SQLDelete` 어노테이션에 명시된 쿼리로 나간다.)
3. 부모엔티티(Team)에 Delete 쿼리가 나가게 됩니다. (`@SQLDelete` 어노테이션에 명시된 쿼리로 나간다.)

- 1번이 예상치 못한 쿼리로 나가게 되는것 입니다.

> 일대다 조인컬럼 방식에서 children.remove(child)를 실행해서 children 쪽의 레코드 삭제를 시도하면 실제 쿼리는 delete가 아니라 해당 레코드의 parent_id에 null을 저장하는 update가 실행된다. 의도와 다르게 동작한 것 같아서 이상해보이지만, 일대다 단방향 매핑에서 children.remove(child)는 사실 child 자체를 삭제하라는 게 아니라 child가 parent의 children의 하나로 존재하는 관계를 remove 하라는 것이다. 따라서 child 자체를 delete 하는 게 아니라 parent_id에 null 값을 넣는 update를 실행하는 게 정확히 맞다. 이 부분의 코드도 신동민 님이 알려주셨는데 [여기](https://github.com/hibernate/hibernate-orm/blob/master/hibernate-core/src/main/java/org/hibernate/persister/collection/OneToManyPersister.java)에서 확인할 수 있다. 
[Reference](https://homoefficio.github.io/2019/04/28/JPA-%EC%9D%BC%EB%8C%80%EB%8B%A4-%EB%8B%A8%EB%B0%A9%ED%96%A5-%EB%A7%A4%ED%95%91-%EC%9E%98%EB%AA%BB-%EC%82%AC%EC%9A%A9%ED%95%98%EB%A9%B4-%EB%B2%8C%EC%96%B4%EC%A7%80%EB%8A%94-%EC%9D%BC/)



---

## 양방향 매핑시
> Member(Many) 쪽에서 team_id 를 가지고 있는 상태에서, SoftDelete 된다.

### 변경된 엔티티
```kotlin
@SQLDelete(sql = "UPDATE team SET deleted_at = NOW() WHERE team_id = ?")
@Where(clause = "deleted_at is null")
@Table(name = "team")
@Entity
class Team(
    @Column(name = "name")
    val name: String,

    // 연관관계 주인은 MyMember(fk team_id 소유)
    @OneToMany(mappedBy = "team", cascade = [CascadeType.ALL], orphanRemoval = true)
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


@Where(clause = "deleted_at is null")
@SQLDelete(sql = "UPDATE my_member SET deleted_at = NOW() WHERE member_id = ?")
@Table(name = "my_member")
@Entity
class MyMember(
    val name: String,

    // 양방향 매핑
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "team_id")
    var team: Team,

    @Column(name = "deleted_at")
    var deletedAt: ZonedDateTime? = null,

    @Column(name = "member_id")
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long = 0L
)
```


```kotlin
@Entity
class Team(
    ...

    // 연관관계 주인은 MyMember(fk team_id 소유)
    @OneToMany(mappedBy = "team", cascade = [CascadeType.ALL], orphanRemoval = true)
    val members: MutableList<MyMember> = mutableListOf(),

    ...
) {
    fun addMember(member: MyMember) {
        members.add(member)
        member.team = this
    }
}

@Entity
class MyMember(
    ...

    // 양방향 매핑
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "team_id")
    var team: Team,
)
```


![image](https://user-images.githubusercontent.com/43930419/232774482-f7830d6c-a7c5-4b92-8efb-ffda8c6bf123.png)

- Team 삭제 요청시, 양방향 연관관계라서 FK 를 들고 있는 연관관계의 주인(MyMember) 를 바로 찾을수 있습니다.
- MyMember 삭제 쿼리 -> Team 삭제쿼리 이렇게 의도한대로 2번만 나가게 됩니다.
- 저장시에도 null 로 채우고 업데이트 하지않고, 바로 insert 쿼리가 나가게 됩니다.
- 이런 경우를 대비하기 위해서, 어쩔수 없이 **매핑 자체는 양방향**을 사용해야하지만, **MyMember 쪽에는 로직을 넣지 않게 하여** 단방향 연관관계처럼 사용할 수 있습니다.

---

## Conclusion
- `OneToMany` 단방향 매핑은 FK 위치 문제 때문에, update 쿼리가 또 한번 나가게 된다.
- 성능차이는 미비하나 쿼리를 보았을때 개발자가 헷갈릴 여부가 있고, soft delete 시 히스토리 관리가 잘 안되기 때문에, 단방향 관계의 경우에도 매핑자체는 양방향 매핑관계를 사용한뒤 Many 쪽에 로직을 두지 않는식으로 개발을 진행하자.

---

## Reference
- https://github.com/hibernate/hibernate-orm/blob/main/hibernate-core/src/main/java/org/hibernate/persister/collection/OneToManyPersister.java
- https://homoefficio.github.io/2019/04/28/JPA-%EC%9D%BC%EB%8C%80%EB%8B%A4-%EB%8B%A8%EB%B0%A9%ED%96%A5-%EB%A7%A4%ED%95%91-%EC%9E%98%EB%AA%BB-%EC%82%AC%EC%9A%A9%ED%95%98%EB%A9%B4-%EB%B2%8C%EC%96%B4%EC%A7%80%EB%8A%94-%EC%9D%BC/
- https://dublin-java.tistory.com/51
- https://ocblog.tistory.com/70
- https://www.inflearn.com/questions/201567