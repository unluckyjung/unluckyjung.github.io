---
title: JPA 사용시 ZoneDateTime을 Audit 하여 자동으로 시간을 넣어주기
date: 2022-05-16-01:50
categories:
- JPA

tags:
- JPA
- Audity
- ZoneDateTime

---

## JPA에서 ZoneDateTime을 Aduitting 해봅니다.
> `@PrePersist`, `@PreUpdate` 와 같은 엔티티 생명주기 콜백과 관련된 어노테이션을 이용합니다.

---

## Problem
> ZoneDateTime 미지원  

- [data-jpa 공식 레포 이슈](https://github.com/spring-projects/spring-data-jpa/issues/1579) 에 따르면 srping data jpa에서 사용하는 `@CreatedDate` `@LasteModifiedDate`는 **ZoneDateTime** 을 지원하지 않습니다.


```kotlin
@MappedSuperclass
@EntityListeners(AuditingEntityListener::class)
abstract class BaseEntity(
    @Column(name = "created_at")
    @CreatedDate    // 에러발생
    var createdAt: ZonedDateTime = ZonedDateTime.now(),

    @Column(name = "updated_at")
    @LastModifiedDate   // 에러발생
    var updatedAt: ZonedDateTime = ZonedDateTime.now()
)
```

> Supported types are [org.joda.time.DateTime, org.joda.time.LocalDateTime, java.util.Date, java.lang.Long, long].; nested exception is java.lang.IllegalArgumentException:


- **ZonedDateTime** 타입의 형태로 **AuditingEntityListener** 를 이용하여 시간을 정해주려고 할경우 위와 같은 에러가 발생합니다.
- `Invalid date type class` 로그를 보고, 타입이 지원되지 않는것을 확인할 수 있습니다.

---

## @PrePersist 와 @PreUpdate 를 이용합니다.
> JPA Entity의 라이프 사이클을 관리하는 어노테이션을 이용합니다.



```kotlin
@MappedSuperclass
abstract class BaseEntity(
    @Column(name = "created_at")
    var createdAt: ZonedDateTime = ZonedDateTime.now(),

    @Column(name = "updated_at")
    var updatedAt: ZonedDateTime = ZonedDateTime.now()
) {
    @PrePersist
    fun prePersist() {
        createdAt = ZonedDateTime.now()
        updatedAt = ZonedDateTime.now()
    }

    @PreUpdate
    fun preUpdate() {
        updatedAt = ZonedDateTime.now()
    }
}
```

- `@PrePersist`: before persist is called for a new entity
- `@PreUpdate`: before the update operation

- 두개의 어노테이션을 이용하면 생성되는 시간과, 업데이트 되는 시간을 정해줄 수 있게 됩니다.


---

## Conclusion
- Spring Data JPA사용시 **ZonedDateTime** 에는 `@CreatedDate` 와 `@LasteModifiedDate` 를 적용할 수 없다.
- 엔티티 생명주기에 맞춰 로직을 넣는 `@PrePersist` 와 `@PreUpdate` 를 이용해 시간을 넣어줄 수 있다.

---

## Reference
- https://github.com/spring-projects/spring-data-jpa/issues/1579
- https://www.baeldung.com/jpa-entity-lifecycle-events#lifecycle
- https://gist.github.com/dlxotn216/94c34a2debf848396cf82a7f21a32abe
- https://logical-code.tistory.com/173