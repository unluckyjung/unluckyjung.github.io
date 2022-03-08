---
title: Optional orElse vs orElseGet
date: 2022-03-09-02:06
categories:
- Java

tags:
- Java

---

## Optional orElse vs orElseGet
> Optional의 orElse와 orElseGet의 차이를 알아봅니다.

## Goal
- Optional의 orElse와 orElseGet의 차이를 알아봅니다.
- 특히 JPA를 사용하는 경우 주의할점을 알아봅니다.

---

## null 일때의 동작이 다릅니다.
> `OrElse`는 항상 동작합니다. `orElseGet`은 null인 경우에만 동작 합니다.

![image](https://user-images.githubusercontent.com/43930419/155703889-2956f9cc-df8c-4241-b35c-5bbf64796597.png)

[docs](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html#orElse-T-)에도 써있는 내용이지만, 실제 메소드의 구현체를 한번 보겠습니다.  


- 여기서 유심히 봐야하는것은, 인자와 반환값입니다.
- `orElse`의 경우 반환해야할값을 인자로 그냥 받지만, `orElseGet`의 경우 **Supplier**로 한번더 wrapping 된것을 인자로 받습니다.
- `orElse`의 경우에는 반환값이 객체이므로, **들어온 메소드**를 일단 실행 시켜버립니다. 
- `orElseGet`의 경우에는 **Supplier** 형태이므로 인자로 들어온 객체가 null인 경우에만, `get()` 을 호출해 생성된 값을 반환합니다.

```java
memberRepository.findByName(name).orElse(saveMember(name));
memberRepository.findByName(name).orElseGet(saveMember(name));
```

- 밑에서 설명할 예제를 미리 살짝 보겠습니다.
- `orElse`, `orElseGet` 둘다 넣어주는 것은 **method** 입니다.
- 하지만 `orElse`의 경우는 반환해야하는 타입과, 인자 타입이 객체이므로 일단 메소드를 실행시켜 버리게 됩니다.


---

## JPA를 사용할때 조심해야할점
> find시 Optional 형태의 값을 반환받게 되므로 주의해야합니다.  

```java
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Entity
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    public Member(final String name) {
        this.name = name;
    }
}

public interface MemberRepository extends JpaRepository<Member, Long> {
    Optional<Member> findByName(final String name);
}

```

```java
@DataJpaTest
@Slf4j
class MemberRepositoryTest {

    @Autowired
    private MemberRepository memberRepository;

    @BeforeEach
    void setUp() {
        memberRepository.save(new Member("unluckyjung"));
    }

    @Test
    void memberGetTest() {
        Member member = getMember("unluckyjung");

        //..추가적인 비즈니스 로직..

        assertThat(memberRepository.findAll().size()).isEqualTo(1);
    }

    private Member getMember(final String name) {
        return memberRepository.findByName(name).orElse(saveMember(name));
    }

    private Member saveMember(final String name) {
        log.info("saveMember run!!");
        return memberRepository.save(new Member(name));
    }
}
```

> `getMember()` 에서는 인자로 들어온 이름을 가진 멤버가 있다면 바로 불러오고, **해당 이름을 가진 유저가 없는 경우** 에 인자로 들어온 이름을 가진 유저를 생성해서 저장하는것을 의도했습니다.


- 실제로는 이미 `setUp()`에서 동일한 이름인 **unluckyjung** 이라는 유저가 있으니 `saveMember가()` 호출되지 않길 기대하겠죠.
- 중복 저장이 되지 않고, 기존의 유저 한개만 계속 DB에 저장될테니 `findAll()` 시 **1개** 의 유저정보만 불려오는것을 기대합니다. 

![image](https://user-images.githubusercontent.com/43930419/155710695-fa93fee3-e64e-480d-893d-3cf3323464c4.png)

![image](https://user-images.githubusercontent.com/43930419/155710787-ad0271d5-00d8-4834-8e46-d1db1395604d.png)

- 하지만 테스트 코드를 돌려보면 1개가 아닌 **2개의 Member**가 저장되어있어 테스트에 실패하고, 로그를 통해 `saveMember()` 메서드가 호출되는것도 알수 있습니다.

- **의도했던것과 완전히 다르게 작동하는것이죠.**

> 의도한대로 수정하는 방법은 아주 간단합니다.

```java
    private Member getMember(final String name) {
        // return memberRepository.findByName(name).orElse(saveMember(name));
       return memberRepository.findByName(name).orElseGet(() -> saveMember(name));
    }
```

- `orElseGet()` 으로 바꿔주어 null인 경우에만 호출하도록 하면됩니다.
- 이 예제를 통해서 알수 있는것은 `orElse`를 사용하는경우 비즈니스 로직에 버그를 유발할 가능성이 있고, **항상 메소드가 호출되니 성능적인 측면**에서도 손해가 있을수 있다는것을 알 수 있습니다.

---

## Conclusion
- `orElse`는 Optional에 담겨있는값이 **null이 아닌 경우에도 메소드를 호출합니다.**
- `orElse`를 사용함으로 인해 일어날수 있는 버그나, 성능손해를 경계해야합니다. **(특히 JPA 사용시)**

---

## Code
- 해당 코드들은 전부 [github](https://github.com/unluckyjung/blog-codes/tree/main/optional-orElse-orElseGet)에서 보실 수 있습니다. 

---

## Reference
- https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html#orElse-T-
- https://www.baeldung.com/java-optional-or-else-vs-or-else-get
- https://stackoverflow.com/questions/33170109/difference-between-optional-orelse-and-optional-orelseget