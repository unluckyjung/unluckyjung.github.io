---
title: 맥북에서 Java 버전을 변경하기
date: 2021-12-15-23:00
categories:
- Dev

tags:
- Java
- Mac
- Setting

---

## 맥북에서 Java 버전을 변경해봅니다.
> 자꾸 까먹어서 기록해둡니다.

---

## Goal
- 자바 버전을 변경해봅니다.
- 자바 버전 기본값을 설정해봅니다.

---

## JDK 버전 확인, 변경

```
-- 현재 자바 버전 확인
java -version   

-- 설치되어있는 버전 확인
/usr/libexec/java_home -V
```

![image](https://user-images.githubusercontent.com/43930419/146204586-ffc00ad2-209a-4f67-b0cc-145c34f0ebdd.png)

- 저의 경우 현재 `1.8.0` 버전을 사용중이고, `11.0.11` 버전도 같이 설치되어있는것을 확인할 수 있습니다.


> 변경

```
-- 1.8 버전으로 변경
export JAVA_HOME=$(/usr/libexec/java_home -v 1.8)

- 11 버전으로 변경
export JAVA_HOME=$(/usr/libexec/java_home -v 11)

-- 변경된 자바 버전 확인
java -version   
```

---

## 자바 기본 버전 설정
> 해당 작업을 생략시, 재부팅 때마다 변경하기전의 자바 버전으롤 롤백됩니다.

```
-- zsh 사용자
vi ~/.zshrc

-- bash 사용자
vi ~/.bash_profile
```

- 저의 경우에는 zsh 사용자이므로 `~/.zshrc` 에 접근합니다.

![image](https://user-images.githubusercontent.com/43930419/146205780-a99f1bdb-cf5c-4ecb-863c-3cc24670b176.png)

```
-- 설정 추가
export JAVA_HOME=$(/usr/libexec/java_home -v 1.8) 
```

- 아랫줄에 버전을 변경할때 사용했던 명령을 붙여 넣어줍니다.

```
source ~/.zshrc
```

- 스크립트를 실행해 적용해줍니다.