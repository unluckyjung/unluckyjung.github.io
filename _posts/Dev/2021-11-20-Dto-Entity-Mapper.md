---
title: MapStruct 라이브러리를 이용해 DTO <-> Entity 변환 하기
date: 2021-11-20-18:00
categories:
- Dev

tags:
- Java
- Library

---

## Dto 와 Entity 변환을 라이브러리를 이용해 손쉽게 해봅니다.
> MapStruct 를 이용합니다.

---

## Goal
- MapStruct 라이브러리를 이용해 `DTO <-> Entity` 변환을 해봅니다.

---

## 의존성 추가

```java
    annotationProcessor 'org.mapstruct:mapstruct-processor:1.4.2.Final'
    implementation 'org.mapstruct:mapstruct:1.4.2.Final'
```

### 전체 Gradle

```java
plugins {
    id 'java'
}

group 'org.example'
version '1.0-SNAPSHOT'

repositories {
    mavenCentral()
}

dependencies {

    compileOnly 'org.projectlombok:lombok:1.18.22'

    annotationProcessor 'org.projectlombok:lombok:1.18.22'
    annotationProcessor 'org.mapstruct:mapstruct-processor:1.4.2.Final'

    implementation 'org.mapstruct:mapstruct:1.4.2.Final'

    testImplementation 'org.junit.jupiter:junit-jupiter:5.7.0'
    testImplementation 'org.assertj:assertj-core:3.19.0'

    testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine:5.7.0'

    testCompileOnly 'org.projectlombok:lombok:1.18.22'

    testAnnotationProcessor 'org.projectlombok:lombok:1.18.22'
}

test {
    useJUnitPlatform()
}
```

---

## Entity와 DTO 구현
> Member관련 DTO와 Entity 클래스를 만듭니다.

```java
@Builder
@Getter
@AllArgsConstructor
public class MemberEntity {

    private Long id;
    private String name;
    private int age;
}
```

```java
@Builder
@Getter
@AllArgsConstructor
public class MemberDTO {

    private String name;
    private int age;
}
```

- From 쪽은 반드시 **Getter**를 만들어야 합니다. (Entity to DTO의 경우 From은 Entity가 됩니다.)
- ~~To 쪽은 반드시 **Setter** 혹은 **Builder**를 만들어야 합니다.~~
- 제공되는 생성자만으로 인스턴스 생성에 문제가 없다면, **Setter**, **Builder**가 없어도 사용가능합니다.

- 먼저 롬복을 이용해 **Builder**를 만들어주었습니다.
- 둘의 차이가 있다면 DTO의 경우에는 id가 없지만, Entity에는 id가 있습니다.

---

## Entity Mapper 인터페이스 구현

```java
public interface EntityMapper<D, E> {

    E toEntity(final D dto);

    D toDto(final E entity);
}
```

- Entity와 DTO로 변환시켜줄 상위 인터페이스를 만들어주었습니다.

```java
@Mapper // 어노테이션을 붙여줍니다.
인터페이스<DTO, Entity> {
    인테페이스명 MAPPER = Mappers.getMapper(인터페이스명.class);

    @Mapping(taget = 필드명, constant = 디폴트값) // 필드명과, 디폴트값은 string 형태로 해주어야 합니다. 
    메소드() ;
}
```

```java
@Mapper
public interface MemberMapper extends EntityMapper<MemberDTO, MemberEntity> {

    MemberMapper MAPPER = Mappers.getMapper(MemberMapper.class);

    @Override
    @Mapping(target = "id", constant = "777L")
    MemberEntity toEntity(final MemberDTO dto);
}
```

- 본체는 `MemberMapper` 인터페이스 입니다.
- 오버라이딩 한부분은, 디폴트로 값을 넣어주고 싶은 경우에 사용합니다.
  - 해당 부분을 생략한다면, **DTO to Entity**과정에서 id값에는 null값이 들어가게 됩니다.

---

## 사용법

```java
MemberEntity memberEntity = MemberMapper.MAPPER.toEntity(memberDTO);
MemberDTO memberDTO = MemberMapper.MAPPER.toDto(memberEntity);
```

- 위처럼 사용하면 `DTO <-> Entity` 변환이 자유롭게 됩니다.

---

## 테스트

```java
class MemberEntityTest {

    @DisplayName("id값 없이 DTO를 가지고 Entity를 만들면, id는 777로 Entity가 만들어진다.")
    @Test
    void toEntityTest() {
        //given
        MemberDTO memberDTO = new MemberDTO("yoonsung", 29);

        //when
        MemberEntity memberEntity = MemberMapper.MAPPER.toEntity(memberDTO);

        //then
        MemberEntity expectedEntity = new MemberEntity(777L, "yoonsung", 29);
        assertThat(memberEntity).usingRecursiveComparison().isEqualTo(expectedEntity);
    }


    @Test
    void toDtoTest() {
        //given
        MemberEntity memberEntity = new MemberEntity(1L, "yoonsung", 30);

        //when
        MemberDTO memberDTO = MemberMapper.MAPPER.toDto(memberEntity);

        //then
        MemberDTO expectedDTO = new MemberDTO("yoonsung", 30);
        assertThat(memberDTO).usingRecursiveComparison().isEqualTo(expectedDTO);
    }
}
```

- 동등성을 검증하는데 사용한 `usingRecursiveComparison()`에 관한 내용은 [이곳](https://unluckyjung.github.io/testcode/2021/06/27/Recursive-Compare/)에서 확인하실 수 있습니다.


### 실행 결과
> 테스트 통과

![image](https://user-images.githubusercontent.com/43930419/142632542-e031a97f-a6f8-459a-a54d-2942694905ed.png)
![image](https://user-images.githubusercontent.com/43930419/142632746-492aa75b-15e8-4566-87b6-f185af85d27e.png)

- `Gradle` 빌드를 했다면, **build**에 `MemberMapperImpl` 라는 이름으로 실제코드가 자동으로 구현된것을 확인할 수 있습니다.
  - `IntelliJ` 빌드를 했다면, **out**에서 확인하실 수 있습니다.

- `@Builder` 어노테이션을 사용했기 때문에, 빌더를 이용해 구현되는것도 확인할 수 있습니다.



<br>

---

## @Builder 를 사용하지 않아도, 사용이 가능합니다.

```java
// @Builder
@Getter
@AllArgsConstructor
public class MemberEntity {

    private Long id;
    private String name;
    private int age;
}
```

```java
// @Builder
@Getter
@AllArgsConstructor
public class MemberDTO {

    private String name;
    private int age;
}
```

![image](https://user-images.githubusercontent.com/43930419/142719557-ab217303-45ac-4edc-9912-4f4a6760ef5c.png)

- `@Builder` 어노테이션을 제거한경우, **생성자에 주입**해주며 변환작업을 하는 코드가 만들어지는것을 확인할 수 있습니다.


<br>

---


## DTO와 Entity의 형태가 같은 경우
> 메소드에 `@Mapping` 어노테이션을 따로 달아주지 않아도 됩니다.

```java
@Getter
@AllArgsConstructor
public class TeamEntity {

    private String name;
    private String color;
}
```

```java
@Getter
@AllArgsConstructor
public class TeamDTO {

    private String name;
    private String color;
}
```

```java
@Mapper
public interface TeamMapper extends EntityMapper<TeamDTO, TeamEntity> {

    TeamMapper MAPPER = Mappers.getMapper(TeamMapper.class);
}
```

### 테스트 코드
> 통과

```java
class TeamEntityTest {

    @Test
    void toEntityTest() {

        //given
        TeamDTO teamDTO = new TeamDTO("team1", "red");

        //when
        TeamEntity teamEntity = TeamMapper.MAPPER.toEntity(teamDTO);

        //then
        assertThat(teamDTO).usingRecursiveComparison().isEqualTo(teamEntity);
    }
}
```

---

## DTO와 Entity의 필드명이 다른경우

```java

//DTO to Entity
@Mapping(source = DTO필드명, target = Entity필드명)
UserEntity toEntity(final UserDTO dto);
```

- `@Mapping` 어노테이션의 source와 taget 속성을 이용합니다.

```java
@Getter
@AllArgsConstructor
public class UserEntity {
    
    private Long id;
}

...

@Getter
@AllArgsConstructor
public class UserDTO {

    private Long userId;
}
```


```java
@Mapper
public interface UserMapper extends EntityMapper<UserDTO, UserEntity> {

    UserMapper MAPPER = Mappers.getMapper(UserMapper.class);

    @Override
    @Mapping(source = "userId", target = "id")
    UserEntity toEntity(final UserDTO dto);

    @Override
    @Mapping(source = "id", target = "userId")
    UserDTO toDto(final UserEntity entity);
}
```

### 테스트코드
> 통과

```java

class UserEntityTest {

    @Test
    void toEntityTest() {

        //given
        UserDTO userDTO = new UserDTO(777L);

        //when
        UserEntity userEntity = UserMapper.MAPPER.toEntity(userDTO);

        //then
        assertThat(userEntity.getId()).isEqualTo(userDTO.getUserId());
    }
}
```

---

## ETC
- Mapping을 지원해주는 라이브러리는 **MapStruct** 외에도 다양하게 있습니다.
- 하지만 퍼포먼스를 비교한 [자료](https://www.baeldung.com/java-performance-mapping-frameworks#1averagetime)와 점유율을 확인한 [자료](https://meetup.toast.com/posts/213)를 본다면, 저는 **MapStruct** 를 선택하겠습니다.
- 이 이유 외에도 컴파일단에서 오류가 확인되고, 직접 생성된 코드를 확인할 수 있고, 속도에 영향을 받는 리플랙션을 사용하지 않는다는 점에서 **MapStruct**을 사용할 이유는 충분해보입니다.

---

## Code
- 해당코드들은 전부 [github](https://github.com/unluckyjung/blog-codes/tree/main/MapStruct)에서 확인 하실 수 있습니다.

---

## Reference
- https://mapstruct.org/
- https://mapstruct.org/documentation/stable/reference/html/#_gradle
- https://meetup.toast.com/posts/213
- https://www.baeldung.com/java-performance-mapping-frameworks
- https://huisam.tistory.com/entry/mapStruct