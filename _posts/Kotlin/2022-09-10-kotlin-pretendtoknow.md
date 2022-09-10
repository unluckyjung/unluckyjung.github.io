---
title: Kotlin 매우 알은체하기 세미나 정리
date: 2022-09-10-13:00
categories:
- Kotlin
- Spring

tags:
- Kotlin
- Spring
- Seminar

---

## 코틀린 매우 알은체하기
> 우아한 형제들에서 온라인으로 진행했던, 박재성님의 세미나 자료중 핵심적인 내용 + @ 를 정리해봅니다.

---

## 코틀린
> 코틀린은 멀티 플랫폼 언어를 지향한다.

![image](https://user-images.githubusercontent.com/43930419/171986521-877e26bd-f0d2-4a7f-a5ac-382e1209310b.png)

- JVM, 자바스크립트, 네이티브 다 동작하게 할수 있는것이 현재 코틀린 재단의 니즈입니다.

---

## 코틀린 아이템

## 아이템1: 코틀린 표준 라이브러리를 익히고 사용하라.
> 자바의 라이브러리를 import 해서 쓰지말고, 코틀린 표준 라이브러리를 찾아서 공부하고 사용해라.

- 예를들어 Random 기능이 필요한경우, 자바의 경우 Thread Safe 하게 할지를 고민해야하지만, 코틀린에서 제공하는 표준 라이브러리 Random의 경우 **기본적으로 Thread Safe** 하게 제공합니다.
- 코틀린은 읽기 전용 컬렉션과 변경 가능한 컬렉션을 구별해서 제공합니다.

## 아이템2: 자바로 디컴파하는 습관을 들여라
> 익숙한 자바코드로는 어떻게 표현되는지를 확인하라.


- 관련있는 [개인 블로그 게시글](https://unluckyjung.github.io/kotlin/intellij/2022/06/06/kotlin-decompile/)

![image](https://user-images.githubusercontent.com/43930419/171986811-48766102-844a-4e1d-bd76-a767b75eedfd.png)

![image](https://user-images.githubusercontent.com/43930419/171986822-53d41bfb-6399-4322-a180-33b0e44ca775.png)

![image](https://user-images.githubusercontent.com/43930419/171986836-40a0b12a-73cf-4b2a-b778-d36b0aefc647.png)

> `shift x 2` 이후 kotlin byte code 입력이 더 편합니다.


- java -> kotlin 전환시, 잘못된 코드를 짜지 않게 확인해보면서 코드를 작성해보는 습관을 들이는게 좋습니다.

> 코틀린과 Java를 같이 쓸수 있는 이유는 같은 JVM 언어이기 때문이다.


![image](https://user-images.githubusercontent.com/43930419/171986910-9069a67d-0e41-403f-9de1-bc2261b2bdfe.png)

- 자바와 코틀린 코드를 동시에 쓰는경우 작성한 코틀린 코드가 코틀린 컴파일러를 통해서 `class` 파일로 1차로 변환됩니다.
- 이후 변환된 class파일과, 자바로 구현되어있는 코드를 같이 또 다시 컴파일합니다. 이때 자바의 에너테이션 프로세싱이 진행됩니다.

## 아이템3: 롬복 대신 데이터 클래스를 사용하라.
> 데이터 전달이 주 목적인 클래스를 생성하는 경우, 데이터 클래스를 사용하라.


- 코틀린 컴파일 이후, 에너테이션 프로세싱이 진행되기 때문에 롬복을 통해서 생성된 자바코드는 코틀린에서 사용이 불가능합니다.
- 자바를 먼저 컴파일하게 하면 사용이 가능하나, 그러면 자바코드에서 코틀린 코드를 호출 할 수 없게 됩니다.


- 데이터 클래스를 사용하는것을 권장합니다.
  - 많은 코드가 축약됩니다.
  - 하나의 코틀린 파일에 여러개의 클래스 파일을 담아도 부담이 없습니다.
  - `copy()` 를 적절히 사용하면 데이터 클래스를 불변으로 관리할 수 도 있습니다.
  - 다만 `toString()` 도 구현되므로, 순함참조의 가능성이 열려있는 JPA의 Entity 에는 사용하지 않는 것을 권장합니다.


### require를 통해서 검증이 가능합니다.
> if(조건) throw Exception 과 같은 보일러 플레이트를 줄일 수 있습니다.

```kotlin
data class Member(
    val age: Long
) {
    init {
        require(age > 0)
    }
}
```

- validate 함수를 작성하지 않고 `require(조건식)`을 넣어서 생성자 파라미터 검증을 해줄 수 있습니다.
- 잘못된 값이 들어올경우 `IllegalArgumentException` 가 발생합니다.
- 이 외에도 `check`, `assert`(JVM -ea 옵션이 들어간경우에만 동작, 비즈니스 로직에는 사용을 권장하지 않음.) 함수도 있습니다.

---

## Kotlin + Spring Boot

## final 키워드가 붙어서 스프링의 기능을 사용하지 못하는 경우를 주의해라.
> 스프링은 프록시 생성을 위해 상속을 많이 이용합니다.


```kotlin
import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.runApplication

@SpringBootApplication
class CrudAppApplication    // final 해서 문제가 됩니다.

fun main(args: Array<String>) {
    runApplication<CrudAppApplication>(*args)
}
```

- 스프링은 기본적으로 `@Configuration` 클래스에 대한 프록시를 생성합니다.
- 하지만 final한 상태에서는 상속이 불가능하므로, open 키워드를 붙여주어야 스프링 기능을 제대로 사용할 수 있게 됩니다.

### All-open 플러그인을 사용합니다.
> 지정한 애너테이션이 있는 클래스와 모든 멤버에 open 변경자를 추가할 수 있습니다.


```kotlin
plugins {
    ...
    kotlin("plugin.spring") version "1.6.21"
}
```

- **스프링 이니셜라이저로 생성**한 프로젝트의 `gradle.kotlin` 파일을 보면 아마 위와 같은 구문이 적혀 있을것입니다.
- 해당 플러그인을 이용하여, 프록시가 필요한 `@Component`, `@Transactional` 과 같은 스프링관련 어노테이션이 붙은 코드에는 open 변경자를 자동으로 붙이게 됩니다.

```kotlin
allOpen{
    annotation("com.my.Annotation")
}

// JPA 사용시 특정 어노테이션들에 프록시가 필요한 경우 open 키워드가 필요합니다.
allOpen {
    annotation("javax.persistence.Entity")
    annotation("javax.persistence.MappedSupperclass")
}
```

- 추가적으로 open 키워드가 추가적으로 필요한 경우에는 위처럼 `gradle.kotlin` 에 추가적으로 넣어주면 됩니다.

## 아이템4: 필드 주입이 필요하면 지연 초기화를 사용하라.
> 의존성이 주입될 필드를 널이 될수 있는 타입으로 관리하지말라.


```kotlin
@Autowired
private latinit var sample: Sample
```

- `lateinit` 키워드를 이용해 지연 초기화를 사용해줍니다.
- 테스트 코드를 작성하는 경우에 유용합니다.

## 아이템5. 변경 가능성을 제한하라. (w SpringBoot)
> 기본적으로 val로 선언하고, 필요할때 var로 변경하라.


![image](https://user-images.githubusercontent.com/43930419/171990294-f4e9bfc9-906a-4120-a34c-d3a2973bb04e.png)

- `students` 의 경우에도, 실제로 프로퍼티로 가지고 있는 student는 언더바를 붙여서 `_student` 로 관리하고, 접근이 가능한 `student` 는 immutable 하게 해서 내보낼 수 있습니다.
- 기본적으로 코틀린의 경우 프로퍼티의 이름이 중복되고 private한 값들은, cpp나 python 처럼 앞에 언더바를 하나 붙여서 관리하는것이 관례입니다.


```kotlin
private var _table: Map<String, Int>? = null
public val table: Map<String, Int>
    get() {
        if (_table == null) {
            _table = HashMap() // Type parameters are inferred
        }
        return _table ?: throw AssertionError("Set to null by another thread")
    }
```

- [공식 문서](https://kotlinlang.org/docs/properties.html#backing-properties) 에서도 해당 방법은 **Backing properties** 라고 부르고 있습니다.

> On the JVM: Access to private properties with default getters and setters is optimized to avoid function call overhead.

- 실제로 어느정도 최적화를 지원해준다고도 합니다.


## 아이템6. 엔티티에 데이터 클래스 사용을 피하라. (w JPA)
> 양방향 연관관계의 경우 순환참조가 발생합니다.

- `toString()`, `hashCode` 를 호출할때 무한 순환 참조가 발생합니다.


## 아이템7. 영속화 되지 않는 필드는 사용자 지정 getter를 사용하라. (w JPA)
> `@Transient` 를 사용하지 않아도 됩니다.


```kotlin

// AS-IS
@Transient
val fixed: Boolean = # logic

// TO-BE
val fixed: Boolean
    get() = # logic
```

- JPA에 의해서 인스턴스화 될때, 초기화 블록이 호출되지 않습니다.
- 이 때문에 `@Transient`가 붙고 로직이 있는경우, null이 될수 없는 val 타입임에도 불구하고, null이 삽입 될 수 있습니다.
- 그러므로, **사용자 getter** 를 이용하여 로직을 타게 하는 방법을 이용하는것을 권장합니다.
- 실제로 다음과 같은값은 getter만 존재하지 **실제로 필드변수가 존재하지 않기 때문**에, 해당 필드는 영속화 되지 않습니다.


## 아이템8. 널이 될수 있는 타입은 빠르게 제거하라. (w JPA)
> 아이디를 0또는 빈 문자열로 초기화하라, 확장함수를 사용해 반복되는 널 검사를 제거할 수 있다.

- [관련된 블로그 글](https://unluckyjung.github.io/jpa/2022/06/19/JPA-save-isNew#)

- Spring Data JPA에서 기본자료형을 사용하고 아이디를 0 으로 잡는경우, new 냐 아니냐를 판단해서 영속화를 하려고 시도합니다. 
- 즉 id가 0인 경우, 새로운 엔티티라고 판단하고 영속화를 하려고 시도합니다.
- **해당 내용은 코틀린에 국한된것이 아니라, JPA의 `isNew()` 체크과정에서 일어납니다.** 핵심은 코틀린에서 null이 정말 필요하지 않는경우, 굳이 nullable 하게 하여 다루지 않는것입니다.




```kotlin
fun TermRepository.getById(id: Long): Term {
    if (id == 0L) {
        return Term.SINGLE
    }
    return findByIdOrNull(id) ?: throw NoSuchElementException("기수가 존재하지 않습니다. id: $id")
}

interface TermRepository : JpaRepository<Term, Long>
```

- **Repository** 에서도 확장 함수를 이용해, 서비스에서 하는 중복되는 null check 작업을 레포지토리로 넣을 수 있게 됩니다.
- 다만 터트리고 싶은 예외가 서비스마다 다를수도 있으므로 고민은 해보아야합니다.

---

## Persistence

## No-arg 컴파일러 플러그인
> JPA 엔티티에 필요한 기본 생성자를 만들어줍니다.


```kotlin
plugins {
    ...
    kotlin("plugin.jpa") version "1.6.21"
    ...
}
```

- `@Entity`, `@Embeddable`, `@MappedSuperclass` 에 기본적으로 적용됩니다.
- 이니셜라이저로 생성시 자동으로 추가됩니다.


---

## TIP

## 잭슨 코틀린 모듈
> 매개변수가 없는 생성자가 없어도 직렬화, 역직렬화가 가능합니다.

```kotlin
dependencies {
    ...
    implementation("com.fasterxml.jackson.module:jackson-module-kotlin")
    implementation("org.jetbrains.kotlin:kotlin-reflect")
    implementation("org.jetbrains.kotlin:kotlin-stdlib-jdk8")
    ...
}
```

- 스프링 이니셜라이저르 생성된 프로젝트에는 해당 의존성들이 기본적으로 포함되어 있습니다.


![image](https://user-images.githubusercontent.com/43930419/171988289-bc46d912-c454-421f-88f7-4c71831a0fe5.png)

- 즉 `ObjectMapper`를 주입을 받는다면, 잭슨 코틀린 모듈이 포함되어있습니다.
- 중요한점은 스프링에서 제공하는 `ObjectMapper` 를 사용하지 않고 유닛테스트를 위해서 직접 `ObjectMapper` 를 인스턴스화 사용하는 경우, 잭슨 코틀린 모듈이 포함되어 있지 않습니다.
- 이러한 경우, `jackSonObjectMapper()` 와 같은 기능을 이용해, 생성해주어야합니다.


![image](https://user-images.githubusercontent.com/43930419/171988487-ac49fc12-b1c6-4cdb-951c-79aa443d08ff.png)


- 예시로  `Jackson2ObjectMapperBuilder.class` 파일의 **853 line** 을 보면, `KotlinDetector` 를 통해서 해당 코드가 코틀린을 사용하고 있는지 확인하고 코틀린 모듈을 등록하고 있는것을 확인할 수 있습니다.
- 위의 의존성들만 주입을 해준다면, 자동으로 코틀린 관련 jackson 모듈을 찾아서 등록해줍니다.
- **주의할점**: `ZoneDateTime` 의 경우에는 `jackSonObjectMapper()` 으로 역직렬화가 불가능하므로, `ObjectMapper` 를 사용해야합니다.


## 코틀린 애너테이션
> 애너테이션 생성 방법을 정확히 지정해줄 수 있습니다.

- [관련된 블로그 글](https://unluckyjung.github.io/kotlin/spring/2022/06/06/kotlin-validation-annotation/)

![image](https://user-images.githubusercontent.com/43930419/171989630-c961e7e3-0a9e-4809-bbaf-d6a65d34e854.png)

- `@JsonProperty` 의 경우 자바로 디컴파일 하면 **필드에 에너테이션**이 붙게 되어버립니다.
- 만약 생성자 파라미터나, getter 를 통해서 애너테이션을 동작시키고 싶은경우에 문제가 됩니다.
- `@params:` `@get:` 을 통해서, 해당 애너테이션이 붙는 위치를 정해줄 수 있습니다.
- 사용하는 라이브러리가 생성자 기반인지, getter 기반인지를 확인하고 에너테이션을 어디에 붙일지를 결정하시면 되겠습니다.


---

## Sample Code

## gradle.kotlin
> 큰따음표를 사용합니다.

- 코틀린 DSL을 이용해, 문법상 오류가 있는지 컴파일타임에 확인할수 있습니다.
- 마우스를 올려서 관련 Docs 에 빠르게 접근 할 수 있습니다.


## DSL

```kotlin
import org.springframework.security.config.annotation.web.builders.HttpSecurity
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter
import org.springframework.security.config.web.servlet.invoke

@EnableWebSecurity
class SecurityConfig : WebSecurityConfigurerAdapter() {
    override fun configure(http: HttpSecurity) {
        http {
            headers {
                frameOptions { disable() }
            }
            csrf { disable() }
            authorizeRequests {
                authorize("/admin/**", authenticated)
                authorize("/**", permitAll)
            }
            formLogin {
                loginPage = "/admin/login"
                permitAll = true
                loginProcessingUrl = "/admin/login"
                failureUrl = "/admin/login?error"
                defaultSuccessUrl("/admin", false)
            }
            logout {
                logoutSuccessUrl = "/admin/login"
            }
        }
    }
}
```

- 위와같이 시큐리티에 관련된 코드를 스프링 진영해서 Kotlin DSL로 지원해주는것을 확인할 수 있습니다.

```kotlin
@Test
fun `올바른 지원서 요청에 정상적으로 응답한다`() {
    every { applicationFormService.getApplicationForm(any(), any()) } returns applicationFormResponse

    mockMvc.get("/api/application-forms") {
        param("recruitmentId", "1")
        header(AUTHORIZATION, "Bearer valid_token")
    }.andExpect {
        status { isOk }
        content { json(objectMapper.writeValueAsString(ApiResponse.success(applicationFormResponse))) }
    }
}
```

- `mockMvc` 를 이용한 테스트 코드의 경우에도 위와같이 코틀린 DSL을 사용하고 있음을 확인할 수 있습니다.

```kotlin
every { evaluationItemRepository.findByEvaluationIdOrderByPosition(any()) } returns evaluationItems
every { evaluationTargetService.gradeAll(any(), any()) } just Runs

assertDoesNotThrow { evaluationTargetCsvService.updateTarget(inputStream, 1L) }
verify(exactly = 1) { evaluationTargetService.gradeAll(any(), any()) }
```

- 만약 void를 반환하게 하고 싶은경우 `just Runs` 를 해주면 됩니다.

---

## Reference
- https://youtu.be/ewBri47JWII