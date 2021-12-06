---
title: 운영체제 차이로 인해 gradlew 사용시 발생하는 No such file or directory 에러 해결
date: 2021-12-06-16:00
categories:
- Dev

tags:
- Gradle
- Window

---

## env: sh/r: No such file or directory 에러를 해결해봅니다.
> Mac과 Window의 개행 표현 차이

## Goal
- `Error with gradlew: /usr/bin/env: bash: No such file or directory`
- 운영체제 차이로 인해 MAC에서 gradlew를 사용시 발생하는 에러를 해결 해봅니다.

## 개요
- clone한 프로젝트를 `gradlew -v` 를 통해서, 프로젝트에서 사용중인 wrapper gradle 버전을 보려는데 문제가 발생.
- 해당 프로젝트는 window에서 생성 되었고, gradlew 역시 윈도우에서 생성 되어짐.

```
chomd +x ./gradlew -- 실행 권한추가

./gradlew -v    -- 명령 입력
/usr/bin/env: bash: No such file or directory   -- 에러발생
```

---

## 원인
> 운영체제 차이의 문제  

- **Window**서 만들어진 `gradlew` 파일이라서, 개행문자 표현의 차이때문에 발생했습니다.
- **Window**에서는 개행을 `CRLF(\r\n)` 으로 표현하지만, 제가 사용중인 **MAC**에서는 `LF(\n)` 으로 표현하기 때문에 위와같은 에러메시지가 발생한것입니다.

## 해결
- `dos2unix` 를 이용해서 개행처리를 해줍니다.

```console
-- homebrew가 설치 되어있는 경우
brew install dos2unix

-- homebrew가 설치되어 있지 않은경우
sudo apt-get install dos2unix
sudo yum install -y dos2unixdos2unix ./gradlew
```

```
-- gradlew 파일 수정
dos2unix ./gradlew

-- 이전에 실패했던 작업 다시 실행
./gradlew -v
```

---

## Reference
- https://newbedev.com/error-with-gradlew-usr-bin-env-bash-no-such-file-or-directory