---
title: Spring EventPubliser 사용법
date: 2023-09-16-16:30
categories:
- Spring

tags:
- Spring
- Event
- Transaction

---

## Spring EventPubliser 사용, 활용법
> 스프링에서 `ApplicationEventPublisher`, `@TransactionalEventListener` 를 이용하여 Event 기반으로 로직을 분리하는법을 알아봅니다.

---

## Goal
- 스프링에서 앞의 트랜잭션이 끝났을때, 이어지는 작업을 수행시키는 이벤트 처리 방법에 대해서 알아봅니다.
- 트랜잭션 및 스레드에 관해서 고려할점을 알아봅니다.

---

## Code

### 이벤트 퍼플리싱 부분

```kotlin
@Component
class DomainEventPublisher private constructor(
    applicationEventPublisher: ApplicationEventPublisher,
) {
    init {
        DomainEventPublisher.applicationEventPublisher = applicationEventPublisher
    }
    companion object{
        private lateinit var applicationEventPublisher: ApplicationEventPublisher

        fun publishEvent(event: Any){
            applicationEventPublisher.publishEvent(event)
        }
    }
}
```

- `ApplicationEventPublisher` 를 주입받으면서 처리해도 되지만, object 객체를 통해 호출이 편하도록 wrapping 해두었습니다.


### 퍼블리셔 호출부분
> 트랜잭션 안에서 퍼블리셔를 호출해야 합니다.

```kotlin
@Transactional
@Component
class MemberSave(
    private val memberRepository: MemberRepository,
) {
    fun save(req: Member.Request): Member {
        val member = Member(name = req.name).let(memberRepository::save)

        // member 가 저장했을때 이벤트 발생
        DomainEventPublisher.publishEvent(
            MemberSaveEvent(memberId = member.id)
        )

        return member
    }
}
```

- 반드시 **트랜잭션 안에서**, 이벤트를 발생시켜 줍니다.

### 이벤트 처리부분

```kotlin
data class MemberSaveEvent(
    val memberId: Long,
)
```

```kotlin
@Component
class MemberSaveEventHandler(
    private val memberRepository: MemberRepository,
) {
    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)  // 루트 트랜잭션이 종료(커밋) 된후 수행하도록 설정
    @Transactional(propagation = Propagation.REQUIRES_NEW)  // 앞에서 트랜잭션이 완료되어 버렸기 떄문에, 여기서 변경작업을 하려면 새로운 트랜잭션을 열어야 함.
    fun onSave(memberSaveEvent: MemberSaveEvent) {

        memberRepository.findByIdOrThrow(memberSaveEvent.memberId).run {
            this.changeName(name = "${this.name} event")
        }
    }
}
```

- 앞의 트랜잭션이 커밋된 이후, 핸들러가 호출될 수 있도록 `@TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)` 를 붙여줍니다.
- 어느 핸들러가 호출될지는 `Argument` 의 타입에 따라 결정되게 됩니다. 
- 여기서는 `MemberSaveEvent` 를 받고 있으므로, 해당 `MemberSaveEvent` 객체가 퍼블리싱 되었을때 앞의 트랜잭션이 종료된후 핸들러가 수행되게 됩니다.
- 중요한것은 `MemberSave` 에서 진행되고 있던 트랜잭션이 종료되고, 해당 `onSave()` 함수가 호출되므로, **트랜잭션이 없는 상태** 이므로 새롭게 트랜재션을 열어주어야합니다.
- 또한 같은 스레드인 경우, `REQUIRES_NEW` 를 사용해주어야 DB 커넥션이 분리되기 때문에 `REQUIRES_NEW` 를 전파레벨로 잡아주어야합니다.

### REQUIREDS_NEWS 를 써야하는 이유
> 스레드 분리가 되지 않는상황에선, 데이터소스 분리가 되지않습니다.

- 이로인해 `@TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)` 를 이용해서 루트 트랜잭션 종료 처리 후
- 이벤트를 잡은뒤 **같은 스레드**에서 트랜잭션을 새로 열어 작업을 처리해주고 싶다면, **반드시** `Propagation.REQUIRES_NEW` 를 사용해주어야합니다.
- 만약 Propagation 을 NEW 로 잡고 싶지 않다면, 이벤트 처리를 하는 로직부터 `MemberSaveEventHandler.onSave` 는 다른 스레드에서 동작을 하도록 코드를 작성해야 합니다.
- 왜냐하면 앞의 트랜잭션이 끝났어도, 새로운 트랜잭션을 명시적으로 생성해주거나 스레드 분리를 하지 않는경우 데이터 소스가 분리가 되지 않아 이벤트 처리부분에서의 기능이 올바르게 동작하지 않기 떄문입니다.

> 저의 경우에는 항상 `REQUIRES_NEW` 로 처리해주고 있긴합니다. 이유는 스레드를 분리시킬시 트랜잭션 데이터소스 문제는 같이 해결되지만, 트랜잭션 분리로 데이터 소스 분리와, 스레드 분리는 좀 별개의 상황이라고 보기 때문입니다. (목적이 불분명할때 스레드 분리로 해결되는것은 얻어걸리는 거라고 생각합니다. 나중에 이걸모르고있다가 동기로직으로 바꾸는데 NEW를 달지 않으면 바로 장애가 나겠죠)


![image](https://github.com/unluckyjung/unluckyjung.github.io/assets/43930419/7327af93-9fad-443e-89cc-baf8c3ce7173)

> NOTE: The transaction will have been committed already, but the transactional resources might still be active and accessible. As a consequence, any data access code triggered at this point will still "participate" in the original transaction, allowing to perform some cleanup (with no commit following anymore!), unless it explicitly declares that it needs to run in a separate transaction. Hence: Use PROPAGATION_REQUIRES_NEW for any transactional operation that is called from here.

- spring [관련 docs](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/support/TransactionSynchronization.html#afterCommit())(TransactionSynchronization) `REQUIRES_NEW` 를 쓰라고 가이드 하고 있습니다.

---

## 기능 검증 테스트코드

```kotlin
@IntegrationTest
class MemberDomainEventTest(
    val memberSave: MemberSave,
    val memberRepository: MemberRepository,
) {

    @DisplayName("도메인 이벤트 발생시키면, 이벤트 리스너에서 받아서 처리된다.")
    @Test
    fun memberSaveDomainEventTest() {
        val beforeName = "yoonsung"
        val afterName = "$beforeName event"

        TestTransaction.flagForCommit()

        val member = memberSave.save(Member.Request(name = beforeName))
        memberRepository.findByIdOrThrow(member.id).name shouldBe beforeName

        // 트랜잭션 종료, @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT) 수행시킴
        TestTransaction.end()

        val afterMember = memberRepository.findByIdOrThrow(member.id)
        afterMember.name shouldBe afterName

        // 메모리 DB가 아닌경우 새로운 트랜잭션에서의 저장이라, 롤백이 안되므로 명시적으로 삭제해줘야함.
        memberRepository.delete(afterMember)
    }
}
```

- `TestTransaction.flagForCommit()`, `TestTransaction.end()` 를 이용하여 테스트 트랜잭션을 강제로 종료하여, 트랜잭션 분리된 상황을 테스트할 수 있는 코드를 작성했습니다.
- 이부분에 대한 글은 나중에 별개의 글로 좀더 상세히 정리하도록 하겠습니다.

---

## Conclustion
- `ApplicationEventPublisher` 을 이용하여 이벤트 기반처리 로직을 작성할 수 있다.
- `@TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)` 를 이용하여 이벤트 수신후 처리하는 로직을 작성할 수 있다.
- 이벤트 핸들러에서, 트랜잭션이 필요한경우 `Propagation.REQUIRES_NEW` 을 사용해 새로운 트랜잭션을 열어주어야한다. 

---

## Reference
- https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/support/TransactionSynchronization.html#afterCommit()
- https://suhwan.dev/2020/01/16/spring-transaction-common-mistakes/
- https://findstar.pe.kr/2022/09/17/points-to-consider-when-using-the-Spring-Events-feature/