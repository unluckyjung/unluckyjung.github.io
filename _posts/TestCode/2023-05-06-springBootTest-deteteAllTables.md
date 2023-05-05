---
title: 테스트 격리를 위해 SpringBootTest 수행중 리스너를 통한 데이터 삭제기능 구현
date: 2023-05-05-16:10
categories:
- TestCode
- Kotlin
- Spring

tags:
- Kotlin
- Spring
- TestCode

---

## SpringBootTest 관련 구동시, 원하는 기능을 테스트 생명주기에 포함시켜 구동시키는 방법을 알아봅니다.
> 테스트 격리를 위한 테이블 전체 삭제 기능을 리스너를 통해 포함시켜봅니다.


## Goal
- `AbstractTestExecutionListener` 를 이용해 테스트 시작전 모든데이터를 지우는 기능을 구현해 봅니다.

---

## 해당 기능이 필요한 시나리오 예시
> 테스트 후 데이터가 남는 경우

- 통합테스트 작성을 하다보면
  - 테스트 에서 트랜잭션 어노테이션을 사용하지 못하거나(테스트 전체가 한개의 트랜잭션으로 묶이면 올바른 테스트 시나리오가 아닌 상황) 
  - 테스트내에서 트랜잭션이 분리되는 등의 이유 (required_new) 로 잔여데이터가 남는 경우가 있습니다.
- 이 경우에는 테스트 시작전후로 DB 에 테스트 관련 데이터를 삭제하여, 테스트 격리환경을 구성해야 합니다.
- 일반적으로 가장 쉬운 방법은 모든 데이터를 삭제해주는 역할을 지닌 부모 클래스를 만든뒤, 해당 클래스에서 `@BeforeEach` 같은 작업을 통해 잔여데이터를 삭제해줄 수 있습니다.
- 하지만 해당 방법은, 모든 테스트가 데이터를 삭제하는 역할을 지닌 부모 클래스에 의존적이게 된다는 문제가 있습니다.
- 혹은 `@SQL` 어노테이션을 이용하는 방법도 있지만, 테이블이 추가, 삭제되거나 하는 상황에서 스크립트를 같이 관리해줘야 한다는 관리포인트 확장 이슈가 있습니다. [@Sql에 관련된 포스트글](https://unluckyjung.github.io/testcode/2021/05/08/Independent-Test-Mehod/)
- **이번 글에서는 스프링에서 제공하는 테스트리스너를 통해, 테스트 전후로 데이터를 삭제 시키는 어노테이션을 만드는 법을 알아봅니다.**

---

## 테스트 격리를 위한 데이터 삭제 코드 작성

```kotlin
class IntegrationTestExecuteListener : AbstractTestExecutionListener() 
```


- `AbstractTestExecutionListener` 클래스를 상속한 구현체 클래스를 만들어줍니다.

![image](https://user-images.githubusercontent.com/43930419/236390774-98cdad73-94cf-46e1-8800-4c8129941c92.png)

- `AbstractTestExecutionListener` 내부를 보면 테스트 구동중 원하는 동작을 끼워넣을 수 있도록 오버라이딩 할 수 있는 함수들을 제공해주고 있습니다.


- 저의 경우에는 테스트 시작 전/후 에 데이터를 전부 지우게 하는 기능을 구현하기 위해서 `before/afterTestMethod` 를 사용해 보겠습니다.

---

## 코드

```kotlin
class IntegrationTestExecuteListener : AbstractTestExecutionListener() {

    override fun beforeTestMethod(testContext: TestContext) {
        val jdbcTemplate = getJdbcTemplate(testContext)
        val transactionTemplate = getTransactionTemplate(testContext)

        truncateAllTables(jdbcTemplate, transactionTemplate)
    }

    override fun afterTestMethod(testContext: TestContext) {
        val jdbcTemplate = getJdbcTemplate(testContext)
        val transactionTemplate = getTransactionTemplate(testContext)
        truncateAllTables(jdbcTemplate, transactionTemplate)
    }

    private fun getTransactionTemplate(testContext: TestContext): TransactionTemplate {
        return testContext.applicationContext.getBean(TransactionTemplate::class.java)
    }

    private fun getJdbcTemplate(testContext: TestContext): JdbcTemplate {
        return testContext.applicationContext.getBean(JdbcTemplate::class.java)
    }

    private fun truncateAllTables(jdbcTemplate: JdbcTemplate, transactionTemplate: TransactionTemplate) {
        transactionTemplate.execute(object : TransactionCallbackWithoutResult() {
            override fun doInTransactionWithoutResult(status: TransactionStatus) {
                jdbcTemplate.execute("set FOREIGN_KEY_CHECKS = 0;")
                JdbcTestUtils.deleteFromTables(jdbcTemplate, *getAllTables(jdbcTemplate).toTypedArray())
                jdbcTemplate.execute("set FOREIGN_KEY_CHECKS = 1;")
            }
        })
    }

    private fun getAllTables(jdbcTemplate: JdbcTemplate): List<String> {
        try {
            jdbcTemplate.dataSource?.connection.use { connection ->
                val metaData: DatabaseMetaData = connection!!.metaData
                val tables: MutableList<String> = ArrayList()
                metaData.getTables(null, null, null, arrayOf("TABLE")).use { resultSet ->
                    while (resultSet.next()) {
                        tables.add(resultSet.getString("TABLE_NAME"))
                    }
                }
                return tables.filter { NOT_DELETE_TABLES.contains(it).not() }
            }
        } catch (exception: SQLException) {
            throw IllegalStateException(exception)
        }
    }

    companion object {
        private val NOT_DELETE_TABLES = setOf("flyway_schema_history")
    }
}
```

- 전체코드는 위와 같습니다.

### 코드 설명

```kotlin
private fun getJdbcTemplate(testContext: TestContext): JdbcTemplate {
    return testContext.applicationContext.getBean(JdbcTemplate::class.java)
}
```

- `IntegrationTestExecuteListener` 자체는 빈으로 등록되지 않기 때문에 잔여데이터를 삭제하기 위한 의존성주입을 생성자로 받을수 없고, 동작하고 있는 테스트 컨텍스트내의 데이터를 지워야하기때문에, 위와같은 방법으로 데이터 삭제를 위해 필요한 스프링 빈들을 얻었습니다.

```kotlin
jdbcTemplate.execute("set FOREIGN_KEY_CHECKS = 0;")
JdbcTestUtils.deleteFromTables(jdbcTemplate, *getAllTables(jdbcTemplate).toTypedArray())
jdbcTemplate.execute("set FOREIGN_KEY_CHECKS = 1;")
```

- `JdbcTestUtils` 를 통해서 테스트 코드내에서 모든 테이블들을 삭제하도록 하였습니다.
- FK 가 맺어진 테이블간에는 바로 삭제가 되지 않기 때문에, FK 체크 과정을 테이블 삭제전 끊어주었습니다.

```kotlin
private fun getAllTables(jdbcTemplate: JdbcTemplate): List<String> {
    try {
        jdbcTemplate.dataSource?.connection.use { connection ->
            val metaData: DatabaseMetaData = connection!!.metaData
            val tables: MutableList<String> = ArrayList()
            metaData.getTables(null, null, null, arrayOf("TABLE")).use { resultSet ->
                while (resultSet.next()) {
                    tables.add(resultSet.getString("TABLE_NAME"))
                }
            }
            return tables.filter { NOT_DELETE_TABLES.contains(it).not() }
        }
    } catch (exception: SQLException) {
        throw IllegalStateException(exception)
    }
}

companion object {
    private val NOT_DELETE_TABLES = setOf("flyway_schema_history")
}
```

- 해당 커넥션에서 사용되고 있는 모든 테이블을 찾아내는 로직입니다.
- 이때 만약 삭제되어선 안되는 테이블이 있다면 `NOT_DELETE_TABLES` 으로 제외하도록 하였습니다.


```kotlin
@Target(AnnotationTarget.CLASS)
@Retention(AnnotationRetention.RUNTIME)
@TestExecutionListeners(
    value = [IntegrationTestExecuteListener::class],
    mergeMode = TestExecutionListeners.MergeMode.MERGE_WITH_DEFAULTS
)
annotation class TruncateAllTables
```

- 어노테이션 기반으로 동작할수 있도록 `TruncateAllTables` 이라는 네이밍을 가진 어노테이션을 만들었습니다.
- 해당 어노테이션은 위에서 만든 `IntegrationTestExecuteListener` 를 `value` 를 가지게 해주었습니다.


---

## 기능 검증

```kotlin
@Order(1)
@Component
class InitLoader(
    private val teamRepository: TeamRepository,
) : ApplicationRunner {

    @Transactional
    override fun run(args: ApplicationArguments?) {
        Team("team1").apply {
            this.addMember(MyMember("member1", this))
            this.addMember(MyMember("member2", this))
        }.let(teamRepository::save)

        Team("team2").apply {
            this.addAllMembers(listOf(MyMember("member3", this), MyMember("member4", this)))
        }.let(teamRepository::save)
    }
}
```

- 테스트 시작전 데이터를 미리 넣어둘수 있도록 `ApplicationRunner` 를 이용해 더미데이터를 삽입해주었습니다.

```kotlin
@IntegrationTest
@TruncateAllTables
class TruncateTablesTest(
    private val teamRepository: TeamRepository,
) {
    @Test
    fun truncateTest() {
        teamRepository.findAll() shouldBe emptyList()
    }
}

@IntegrationTest
class TruncateTablesTest2(
    private val teamRepository: TeamRepository,
) {
    @Test
    fun truncateTest() {
        teamRepository.findAll().size shouldNotBe 0
    }
}
```

- `@TruncateAllTables` 이 붙은 테스트는, 테스트 시작전 모든 데이터를 삭제 해주는것을 확인할 수 있습니다.

---

## 참고

> 검증코드에서 사용된 `IntegrationTest` 은 커스텀하게 만들어둔 어노테이션입니다.

```kotlin
@Target(AnnotationTarget.CLASS)
@Retention(AnnotationRetention.RUNTIME)
@TestConstructor(autowireMode = TestConstructor.AutowireMode.ALL)
annotation class TestEnvironment

@Target(AnnotationTarget.CLASS)
@Retention(AnnotationRetention.RUNTIME)
@Transactional
@SpringBootTest
@TestEnvironment
annotation class IntegrationTest
```

> 참고로 본문에는 **SpringBootTest 수행중 리스너를 통한 동작 끼워넣기** 라고 적어두었지만, 스프링 관련 테스트컨텍스트를 사용하고 있는 모든테스트에서도 사용할수 있습니다. ex: `@RepositoryTest` 다만 잔여데이터가 남는 테스트들은 보통 `SpringBootTest` 를 이용한 테스트이기때문에 타이틀을 위와 같이 해두었습니다.

![image](https://user-images.githubusercontent.com/43930419/236393807-46f783a9-0115-41d1-842d-dc62dc6a1247.png)

---

## Github
- 관련된 코드는 [이곳](https://github.com/unluckyjung/kotlin-spring-jpa-playground/tree/test/spring-boot-test/life-cycle)에서 볼 수 있습니다.

---

## Conclusion
- `AbstractTestExecutionListener` 를 이용해 테스트 시작 전후에 원하는 기능을 동작시킬 수 있다.
- 테스트 격리를 위해서, 테스트 시작전/후로 모든 데이터를 삭제하는 기능을 구현해보았다.
  
---

## Reference
- https://www.baeldung.com/spring-testexecutionlistener
- https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/context/support/AbstractTestExecutionListener.html