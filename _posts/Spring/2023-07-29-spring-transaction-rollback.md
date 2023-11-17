---
title: Spring transactional propagation level 에 따른 Rollback 범위 주의
date: 2023-07-29-16:30
categories:
- Spring

tags:
- Spring
- Database
- Transaction

---

## @Transactional 사용시 예상치 못한 롤백이 일어날수 있는 케이스를 알아봅니다.
> 주로 사용하는 `REQUIRED`,`REQUIREDS_NEW` 두가지 사용케이스에 대해서 정리합니다.

## Goal
- 스프링에서 트랜잭션 어노테이션을 이용할때, 예상치 못한 롤백이 일어날 수 있는 상황에 대해서 알아봅니다.
- 롤백이 일어나는 원인을 알아보고, 주의할점과 해결 방법을 알아봅니다.

--- 

## 선결론

### REQUIRES_NEW 사용시 주의할점
> 한개의 스레드에서 동작 한다는것에 주의합니다.

```
tx1 -> tx2 (New) 호출하는 구조인 경우

- tx1 로직 정상수행, tx2 호출
- tx2 에서 예외를 throw

결론: tx1, tx2 작업 전부 롤백 되어짐.
```

- tx2 에서 예외가 발생하고 롤백이 되는 Exception 을 throw 하는경우 tx1, tx2 둘다 롤백됩니다. 
- 가볍게 생각하면 tx2 는 다른 트랜잭션이라 tx1 은 롤백이 안될꺼라 생각할 수 있지만, 결국 tx1 도 tx2랑 같은 스레드이고, throw 된 Exception 때문에 같이 롤백이 진행됩니다.

위 상황의 경우 tx1 에서 tx2 호출하는 부분을 `try-catch` 로 감싸고 예외를 처리해준다면 tx2 만 롤백됩니다.


### REQUIRED 사용시 주의할점
> 같은 트랜잭션이라는 것에 주의합니다.

```
service1(s1) -> service2(s2) 를 호출하는 구조인 경우

- s1 로직 정상수행(tx1), s2 호출(tx1)
- s2 에서 예외를 throw
- s1 에서 s2에서 터진 예외를 catch 하여 추가 throw 하지않음
- s1 에서 나머지 로직 수행

결론: s1, s2 의 tx1 에서 작업한 모든작업이 롤백되어짐
```

- s2 에서 예외가 발생하고, s1 에서 s2 를 try-catch 로 감싸고 있어도 tx1, tx2 둘다 롤백이 진행됩니다.
- 로직은 수행되지만 s1의 커밋 단계에서 `UnexpectedRollbackException` 이 발생하게 됩니다.
- 이유는 s2 에서 던진 Exception 을 보고, **롤백 마크(rollback-only)를 s2 에서 이미 박아 버렸기 때문에**, s1 로직이 종료되고 커밋하려는 시점에서 롤백 마크가 설정되어있어 커밋처리를 못하고 롤백이 진행 되어버립니다.


---

## 동작을 확인하는데 사용될 Code

```kotlin
@Service
class TransactionRollbackTestServiceParent(
    private val memberRepository: MemberRepository,
    private val transactionRollbackTestServiceChild: TransactionRollbackTestServiceChild,
) {

    @Transactional
    fun requiresNew(parentMember: Member, childMember: Member, ex: Exception?) {
        memberRepository.save(parentMember)
        transactionRollbackTestServiceChild.requiresNewSave(childMember, ex)
    }

    @Transactional
    fun requiresNewAndTryCatch(parentMember: Member, childMember: Member, ex: Exception?) {
        memberRepository.save(parentMember)
        try {
            transactionRollbackTestServiceChild.requiresNewSave(childMember, ex)
        } catch (e: Exception) {
            println("Exception caught")
        }
    }

    @Transactional
    fun requiresNewAndThrowParentException(parentMember: Member, childMember: Member, ex: Exception?) {
        memberRepository.save(parentMember)
        transactionRollbackTestServiceChild.requiresNewSave(childMember, null)
        if (ex != null) {
            throw ex
        }
    }

    @Transactional
    fun required(parentMember: Member, childMember: Member, ex: Exception?) {
        memberRepository.save(parentMember)
        transactionRollbackTestServiceChild.requiredSave(childMember, ex)
    }

    @Transactional
    fun requiredAndTryCatch(parentMember: Member, childMember: Member, ex: Exception?) {
        memberRepository.save(parentMember)

        try {
            transactionRollbackTestServiceChild.requiredSave(childMember, ex)
        } catch (e: Exception) {
            println("Exception caught")
        }
        println("after try-catch...")

        memberRepository.save(parentMember)
    }

    @Transactional
    fun requiredAndTryCatchInChild(parentMember: Member, childMember: Member, ex: Exception?) {
        memberRepository.save(parentMember)

        transactionRollbackTestServiceChild.requiredSaveWithTryCatch(childMember, ex)

        memberRepository.save(parentMember)
    }
}

@Service
class TransactionRollbackTestServiceChild(
    private val memberRepository: MemberRepository,
) {
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    fun requiresNewSave(member: Member, ex: Exception?) {
        memberRepository.save(member)

        if (ex != null) {
            throw ex
        }
    }

    @Transactional
    fun requiredSave(member: Member, ex: Exception?) {
        memberRepository.save(member)

        if (ex != null) {
            throw ex
        }
    }

    @Transactional
    fun requiredSaveWithTryCatch(member: Member, ex: Exception?) {
        try {
            memberRepository.save(member)
            if (ex != null) {
                throw ex
            }
        } catch (e: Exception) {
            println("exception throw")
        }
    }
}
```

- AOP 적용을 위해 `TransactionRollbackTestServiceParent` 이 `TransactionRollbackTestServiceChild` 호출하는 서비스 코드를 작성했습니다.
- 모든 코드에 대한 설명보다는 놓칠수 있는 부분에 대한 케이스만, 실제 코드 동작을 통해 같이 확인해보겠습니다.


### REQUIRES_NEW

```kotlin
@DisplayName("[REQUIRES_NEW] 자식 트랜잭션에서 예외가 발생하고, 컨트롤되지 않으면 둘다 롤백된다.")
@Test
fun rollbackTest1() {
    shouldThrowAny {
        transactionRollbackTestServiceParent.requiresNew(
            parentMember = parentMember,
            childMember = childMember,
            ex = IllegalArgumentException(),
        )
    }
    val result = memberRepository.findAll()
    result.size shouldBe 0
}
```

- `Parent` 서비스에서, `Child` 서비스를 호출하면서 새로운 트랜잭션을 여는 케이스입니다.
- `Child` 에서 롤백이 되는 예외를 throw 하고 `Parent` 에서 catch 가 되지 않으면 결국 한개의 스레드이기 때문에, 예외가 계속 해서 전파되고 `Parent`, `Child` 둘다 결과가 롤백되어 저장된 결과가 0인것을 확인할 수 있습니다. (tx1, tx2 둘다 각각 롤백 진행)
  

### REQUIRED
> **사람들이 가장 많이 놓치는 케이스**

```kotlin
@Transactional
fun requiredAndTryCatch(parentMember: Member, childMember: Member, ex: Exception?) {
    memberRepository.save(parentMember)

    try {
        transactionRollbackTestServiceChild.requiredSave(childMember, ex)
    } catch (e: Exception) {
        println("Exception caught")
    }
    println("after try-catch...")

    memberRepository.save(parentMember)
}
```

```kotlin
@DisplayName("[REQUIRED] 하위 서비스에서 예외가 발생하고, 상위 서비스에서 try-catch 예외 처리를 해주어도 UnExpectedRollBackException 이 발생하며 전부 롤백된다.")
@Test
fun rollbackTest5() {
    shouldThrowExactly<UnexpectedRollbackException> {
        transactionRollbackTestServiceParent.requiredAndTryCatch(
            parentMember = parentMember,
            childMember = childMember,
            ex = IllegalArgumentException(),
        )
    }
    val result = memberRepository.findAll()
    result.size shouldBe 0
}
```

- `Child` 서비스에서 예외가 발생하는 경우를 대비해서 `Parent` 입장에서 `child` `try-catch` 를 해주었음에도, 전부다 롤백이 되며 결과가 0인것을 확인할 수 있습니다.
- 또한 트랜잭션이 최초 시자되는 `Parent` 를 호출한 함수에서는 함수 종료 단계에서 최종적으로 `UnexpectedRollbackException` 예외가 발생한것도 같이 확인할 수 있습니다.
- 트랜잭션이 전파레벨에 따라 트랜잭션이 한개로 묶여있는 상황이기 때문에 `child` 에서 `parent` 로 예외를 throw 를 하는 상황에서 `child` 프록시 쪽에서 이미 롤백 마크(rollback-only)가 찍어버리기 때문에, 같은 트랜잭션을 사용하는 `parent` 는 트랜잭션 커밋을 수행하지 못해 본인 로직에서는 예외가 발생한적이 없어도 작업이 전부 롤백됩니다.

<img width="360" alt="image" src="https://github.com/unluckyjung/unluckyjung.github.io/assets/43930419/0e85fbde-d776-4998-a8f3-21afe3aa04ef">

- 여기서 주의할점은 실제로 Parent 입장에서 **로직자체는 전부 수행이 되었다는것** 입니다. (`after try-catch` 출력 까지 호출되어진것을 확인) 만약 롤백이 같이 진행 되어야하는데, API 호출을 해버렸다거나 다른 트랜잭션으로 도는 작업이 있었다면 해당 작업들은 롤백되지 않게되어 데이터 정합성이 깨지게 됩니다.

```kotlin
// Child

@Transactional
fun requiredSaveWithTryCatch(member: Member, ex: Exception?) {
    try {
        memberRepository.save(member)
        if (ex != null) {
            throw ex
        }
    } catch (e: Exception) {
        println("exception throw")
    }
}
```

- 위처럼 child 쪽에서 try-catch 로 잡아준다면, 프록시 입장에서는 예외가 난것을 아예 인지하지 못하기 때문에 **롤백 마크를 박지 않게 되어**, parent 의 작업이 롤백되거나 하는일은 발생하지 않습니다.

```kotlin
// @Transactional 트랜잭션 어노테이션 제거
fun requiredSave(member: Member, ex: Exception?) {
    memberRepository.save(member)

    if (ex != null) {
        throw ex
    }
}
```

- 또 다른 방법으로는 Child 의 함수에서 **트랜잭션 어노테이션을 제거해버렸다면, 롤백이 진행되지 않습니다.**
- 이유는 해당 해당함수에서 AOP 동작을 위한 proxy 가 생기지 않아, 예외를 throw 했다고 해도 트랜잭션을 위한 프록시가 없어 **롤백 마크를 설정하는 과정이 진행되지 않습니다.**
- 다만 이방법의 경우, 반드시 호출하는 쪽에서 트랜잭션을 이미 연 상태 라는것이 보장되어야합니다. (save 호출하는데 트랜잭션이 없으면 예외발생)


### 문제가 되는 케이스

> API 호출을 해버렸다거나 **다른 트랜잭션**으로 도는 작업이 있었다면 해당 작업들은 롤백되지 않게되어 데이터 정합성이 깨지게 됩니다.

```kotlin
@Transactional
fun requiredAndTryCatch2(parentMember: Member, childMember: Member, otherTxChildMember: Member, ex: Exception?) {
    memberRepository.save(parentMember)

    try {
        transactionRollbackTestServiceChild.requiredSave(childMember, ex)
    } catch (e: Exception) {
        println("Exception caught")
    }
    println("after try-catch...")

    transactionRollbackTestServiceChild.
    
    // 새로운 트랜잭션
    requiresNewSave(member = otherTxChildMember)
}
```

```kotlin
@DisplayName("[REQUIRED] UnExpectedRollBackException 이 발생했을때 새로운 트랜잭션에서 작업한 내용은 저장된다.")
@Test
fun rollbackTest5_3() {
    val otherTxChildMemberName = "newTxMember"

    shouldThrowExactly<UnexpectedRollbackException> {
        transactionRollbackTestServiceParent.requiredAndTryCatch2(
            parentMember = parentMember,
            childMember = childMember,
            otherTxChildMember = Member(name = otherTxChildMemberName),
            ex = IllegalArgumentException(),
        )
    }
    val result = memberRepository.findAll()
    result.size shouldBe 1
    result[0].name shouldBe otherTxChildMemberName
}
```

- 위와 같은 경우가 문제가 될 수 있습니다.
- 일부는 롤백이 되고 일부는 롤백이 되지 않는것을 확인할 수 있습니다.

```
- parent 에서 save1(tx1), child 에서 save2(tx1), child 에서 throw exceptoin (tx1)
- tx1 에서 발생한 예외는 try-catch 로 잡았기 때문에 parent 에서 나머지 로직 수행
- child 에서 requires_new 를 통해 다른 트랜잭션에서 save3(tx2)
- tx1 작업은 롤백, tx2 작업은 롤백되지않음.
```

- 이런 케이스가 발생할 수 있기 때문에, 트랜잭션을 올바르게 관리했는지는 항상 잘 고민해 보아야 합니다.
- 예시는 트랜잭션의 분리만 들었지만, 만약 MSA 환경과 같은 형태라서 `UnexpectedRollbackException` 이 발생하는 시나리오에서 로직 수행중 다른 쪽에 Write 성 API 를 쏘게되었다면, 서버간의 정합성을 다시 맞춰야하는 문제가 발생하게 된다는것을 주의해야합니다.

---

## Conclusion
- `REQUIRES_NEW` 사용시 트랜잭션은 분리되지만, 스레드는 같이 때문에, 하위 트랜잭션에서 예외 전파시 롤백이 같이 진행된다.
- `REQUIRED` 사용시 커밋과 같은 작업은 결국 모든 작업이 끝난뒤에 한번에 처리된다. 이때 각 서비스별 트랜잭션이 한개의 트랜잭션으로 묶여있기 때문에, 타 서비스에서 예외가 발생한 경우 Proxy 에서 예외가 터졌다는것을 인지했다면, rollback-only 가 설정되어 전체 작업이 롤백된다.

---

## Code
- 전체 코드는 [Github](https://github.com/unluckyjung/kotlin-spring-jpa-playground/tree/spring/transcation-rollback) 에서 볼 수 있습니다.

---

## Reference
- https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/UnexpectedRollbackException.html
- https://techblog.woowahan.com/2606/