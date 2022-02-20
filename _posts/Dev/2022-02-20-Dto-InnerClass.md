---
title: DTO를 inner class로 관리하기
date: 2022-02-20-23:50
categories:
- Dev

tags:
- Java
- Dto

---

## DTO를 이너 클래스로 관리하기
> DTO를 도메인의 이너 클래스로 관리해, DTO 관리 편의성을 높여봅니다.

---

## Goal
- DTO를 이너클래스로 관리해 개발 편의성을 올려봅니다.
- 도메인 클래스안에 DTO를 이너클래스로 두는것이 설계상 괜찮은지에 대해서 고민해봅니다.

---

## 요청별로 DTO를 외부 클래스로 나누어 관리하는 경우
> 너무 많은 DTO 클래스가 생겨 관리가 힘들어질 수 있습니다.

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
```

- 만약 **Member**라는 도메인 클래스가 있다고 해봅시다.

```java
@Getter
@AllArgsConstructor
@NoArgsConstructor
public class MemberCreateRequest {
    private String name;
}

@Getter
@AllArgsConstructor
public class MemberCreateResponse {
    private Long id;
    private String name;
}
```

- 이때 Member를 create하는 기능이 필요할시, `MemberCreateRequest`, `MemberCreateResponse` 벌써 두개의 DTO 클래스가 필요해집니다.
- 지금은 단순히 2개이지만, 여러 DTO가 계속 필요해지는 상황이 온다면 `MemberxxxRequest` 이런 DTO 클래스가 점점 늘어나게 되어 DTO를 선별하고 구분하는데 인적 리소스가 소모되게 될것입니다.

![image](https://user-images.githubusercontent.com/43930419/154848876-51bf565e-efa7-46e4-9dfb-8235f979fb42.png)  
> 과거 진행한 프로젝트의 수많은 DTO 클래스들...


---


## DTO를 이너 클래스로 관리
> DTO를 관리하는 (인적)비용이 줄어듭니다.  


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

    @Getter
    @AllArgsConstructor
    @NoArgsConstructor
    public static class Request {
        private String name;
    }

    @Getter
    @AllArgsConstructor
    public static class Response {
        private Long id;
        private String name;
    }
}
```

- 위처럼 이너 클래스를 이용해, Member안에 Request와 Response DTO를 가지게 있게 한다면 어떻게 될까요.
- Member 클래스만 보고 관련된 DTO를 빠르게 찾을 수 있게되어, 인적 리소스가 줄어들고 개발의 편의성이 늘어나게 됩니다.

```java
@RestController
@RequestMapping("/api/member")
public class MemberController {
    
    @GetMapping("/{id}")
    public ResponseEntity<Member.Response> getMember(@PathVariable final Long id) {
        return ResponseEntity.ok(new Member.Response(id, "unluckyjung"));
    }

    @PostMapping
    public ResponseEntity<Member.Response> create(@RequestBody final Member.Request request) {
        //...저장 로직
        return ResponseEntity.created(URI.create(String.format("/api/member/%d", 1L)))
                .body(new Member.Response(1L, request.getName()));
    }
}
```

- 이너클래스를 요청, 응답을 처리하는 컨트롤러 예제를 간단하게 작성하면, 위와 같이 구현할 수 있게 됩니다.
- (위 예제가 실제 개발에서 적용되는 방식은 아닙니다. 단순히 이해를 쉽기 위해서 작성한 예제일 뿐입니다.)

---

## 고민되는 사항
> 도메인이 DTO의 형태를 알고 있게 되는건 아닐까? 둘의 결합도가 높아지는건 아닐까?   

- DTO를 도메인 객체와 분리해서 사용하는 이유는, **DTO는 변경이 찾지만 도메인 객체는 그렇지 않습니다.** 따라서 도메인을 DTO로 사용하는것은 변경이 잦은 형태에 의존성을 띠고 있는 형태가 되어, 올바르지 못한 설계가 되기때문에 도메인과 DTO 두개를 분리하는것 입니다.
- 하지만, 위와같이 도메인(엄밀하게 따지면 엔티티이지만, 도메인으로 하겠습니다.) 객체가 DTO를 이너클래스로 가지고 있는 형태는 **도메인이 DTO를 알고 있는 형태가 되어, 잘못된 설계가 아닌가?** 라는 생각이 순간들었습니다.

> **결론적으로는 위의 예시와같은 형태는 괜찮다라고 생각합니다.**  


- 의존성을 띠고 있는 형태라 함은 Domain **객체의 로직에 DTO가 영향**을 주게 되는것을 뜻하는것이라고 생각합니다.
- 하지만 위와 같은 이너클래스 형태는 단순히 도메인 클래스안에 DTO 클래스를 들고만 있지, DTO 와 **도메인의 로직에는 전혀 영향이 없는 형태**입니다. 즉 로직이나 상태에 따른 의존성이 있는 형태는 아닌거죠.
- `도메인 <-> DTO` 끼리 연관이 생긴다는것보다는, 해당 도메인과 관련된 DTO를 같이 묶어두기 위한 정도인거죠.
- 즉, 편의성을 위해서 도메인내에 이너 클래스를 묶어서 사용하는것이기 때문에 다른 시야로 봐야한다고 결론 내렸습니다.
- **물론 여러 도메인들이 합쳐져서 만들어지는 DTO 라면, 한 도메인의 이너클래스로 DTO를 사용하면 안되겠죠.**

### 장점
> 응집도가 높아지는 효과가 있습니다.  


- 위에서 말한 **여러 도메인들이 합쳐져서 만들어지는 DTO 는 이너클래스로 사용하지 않는다.** 라는 룰을 지킨다면
- 개발자 입장에서 이너클래스의 DTO를 보았을때 이너 클래스 DTO는 해당 클래스 안에서만 한정적으로 사용한다는 의미를 부여할 수 있어, 응집력이 높아지고 개발자들이 신경써야 하는 외부클래스의 개수가 줄어들어 개발 편의성을 높이는 효과를 나타낼 수 있다고 생각합니다.

---

## Conclusion
- 이너클래스를 이용해 DTO 관리의 편의성을 높일 수 있다.
- 하지만 이너클래스로 만들어진 DTO는, 속해 있는 도메인 클래스로만 구성되는 경우에 사용해야한다.

---

## Reference
- https://www.inflearn.com/questions/47205
- https://velog.io/@ausg/Spring-Boot%EC%97%90%EC%84%9C-%EA%B9%94%EB%81%94%ED%95%98%EA%B2%8C-DTO-%EA%B4%80%EB%A6%AC%ED%95%98%EA%B8%B0