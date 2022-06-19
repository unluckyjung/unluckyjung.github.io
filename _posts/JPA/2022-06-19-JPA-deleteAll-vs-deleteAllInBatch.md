---
title: JPA deleteAll vs deleteAllInBatch
date: 2022-06-19-16:50
categories:
- JPA

tags:
- JPA

---

## JPA deleteAll vs deleteAllInBatch
> deleteAll 사용시 불필요한 쿼리가 나갈 수 있습니다.

---

## Goal
- JPA에서 제공하는 `deleteAll()` 과 `deleteAllInBatch()` 차이를 알아봅니다.

---

## deleteAll() vs deleteAllInBatch
> 실제로 발생하는 쿼리가 다릅니다.


### deleteAll()
> Select 쿼리가 한번나가고, 삭제 쿼리가 N번 나갑니다.

![image](https://user-images.githubusercontent.com/43930419/174068309-5f852365-3c22-4454-bf86-89277f915d80.png)

![image](https://user-images.githubusercontent.com/43930419/174068450-0724550f-14bd-479a-8781-784bb516490d.png)

- 구현체를 보면 `findAll()` 을 통해서 select을 한번하고, 가져온것을 하나씩 삭제하는것을 확인할 수 있습니다.
- `deleteAllById()` 의 경우에도 아이디 리스트를 받아서 처리하는 겨우 가져온 아이디에 대해서 한번씩 삭제쿼리가 나가는것을 확인할 수 있습니다.
- 만약 10만개의 데이터가 있다면, 조회 1번 + 10만번의 삭제 쿼리가 나가게되어 **성능에 문제가 되는 1 + N 문제가 발생하게 됩니다.**


### deleteAllInBatch()
> 한방 쿼리를 통해서 삭제합니다.


![image](https://user-images.githubusercontent.com/43930419/174068952-98b00752-28d9-4b7a-acbe-c6d3d87b5565.png)


![image](https://user-images.githubusercontent.com/43930419/174038913-2ebb5740-942f-40b6-b832-5af0cd811436.png)

- 엔티티 이름을 기반으로 해당하는 테이블을 찾아서 한방 쿼리로, 데이터를 삭제합니다.

---

## Code

```kotlin
import com.example.kopring.test.RepositoryTest
import org.junit.jupiter.api.BeforeEach
import org.junit.jupiter.api.Test

@RepositoryTest
class MemberRepositoryDeleteTest(
    private val memberRepository: MemberRepository
) {

    @BeforeEach
    internal fun setUp() {
        memberRepository.saveAllAndFlush(
            mutableListOf(
                Member("goodall"),
                Member("unluckyjung"),
                Member("jys"),
            )
        )
    }

    @Test
    fun deleteAll() {
        memberRepository.deleteAll()
        memberRepository.flush()
    }
}
```

```kotlin
select member0_.member_id as member_i1_0_, member0_.created_at as created_2_0_, member0_.updated_at as updated_3_0_, member0_.name as name4_0_ from member member0_
select member0_.member_id as member_i1_0_, member0_.created_at as created_2_0_, member0_.updated_at as updated_3_0_, member0_.name as name4_0_ from member member0_;
Hibernate: delete from member where member_id=?
2022-06-16 21:30:13.578  INFO 4022 --- [           main] p6spy                                    : #1655382613578 | took 1ms | statement | connection 3| url jdbc:h2:mem:d03c6ae9-252b-4af4-8b47-037886375e67
delete from member where member_id=?
delete from member where member_id=1;
Hibernate: delete from member where member_id=?
2022-06-16 21:30:13.580  INFO 4022 --- [           main] p6spy                                    : #1655382613580 | took 0ms | statement | connection 3| url jdbc:h2:mem:d03c6ae9-252b-4af4-8b47-037886375e67
delete from member where member_id=?
delete from member where member_id=2;
Hibernate: delete from member where member_id=?
2022-06-16 21:30:13.581  INFO 4022 --- [           main] p6spy                                    : #1655382613581 | took 0ms | statement | connection 3| url jdbc:h2:mem:d03c6ae9-252b-4af4-8b47-037886375e67
delete from member where member_id=?
delete from member where member_id=3;
2022-06-16 21:30:13.609  INFO 4022 --- [           main] p6spy                                    : #1655382613609 | took 1ms | rollback | connection 3| url jdbc:h2:mem:d03c6ae9-252b-4af4-8b47-037886375e67
```


- 간단한 코드를 작성해서 실제 쿼리를 찍어보면 `deleteAll()` 의 경우에는 1 + N번의 쿼리가 나가는것을 확인할 수 있습니다.


---

## Conclusion
- delete 관련 기능이 필요한경우, 성능을 고려해서 `deleteAllInBatch()` 를 사용하자.


---

## Reference
- https://jessyt.tistory.com/124
- https://blog.yevgnenll.me/posts/jpa-use-not-deleteall-but-deleteallinbatch
- https://stackoverflow.com/questions/26142261/spring-jparepostory-delete-vs-deleteinbatch