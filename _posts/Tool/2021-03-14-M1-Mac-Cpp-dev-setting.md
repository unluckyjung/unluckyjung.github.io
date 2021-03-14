---
title: M1 맥북 cpp 개발환경 세팅하기
date: 2021-03-14-14:00
categories:
- Tool

tags:
- IDE
- Clion

---

## M1 맥북에서 cpp 개발환경을 구축해봅니다.
> clion ide 를 통해


### Goal
- 컴파일러 gcc와 디버거 lldb를 설치한다.
- clion을 설치한다.
- clion에 컴파일러 설정을 해준다.

---


## 터미널을 이용해 xcode를 설치
> g++, gcc, lldb는 같이 자동으로 설치된다.

```
xcode-select --install
```

- 터미널에서 g++, gcc 명령을 입력해 컴파일을 하시는 분들은 이 단계에서 끝입니다.

---


## clion 설치
> 하지만 나는 귀찮은걸 정말 싫어하는 사람이므로, ide를 설치해보겠다.
  

![image](https://user-images.githubusercontent.com/43930419/111025393-1cff0c80-8427-11eb-8d72-bc58207c434d.png)

- [설치링크](https://www.jetbrains.com/ko-kr/clion/) 로 들어가서, `Apple Silicon` 버전의 clion을 다운받아 설치해준다.

---

## 컴파일러 설정
> 현재 clion은 어떤 컴파일러를 통해서 소스파일을 컴파일 해야할지를 모르는 상태이다.

- 이를 해결하기위해서 추가적인 설정이 필요하다.

![image](https://user-images.githubusercontent.com/43930419/111024774-e673c280-8423-11eb-9ec7-4aa11dca5019.png)


- 설정 (단축키 `command + ,`) 으로 들어간다.
  
- ![image](https://user-images.githubusercontent.com/43930419/111024800-fd1a1980-8423-11eb-8018-65cd1af284cd.png)
![image](https://user-images.githubusercontent.com/43930419/111024811-07d4ae80-8424-11eb-81b3-7ded564d3534.png)

- `Build....` 탭을 눌러준다.
- `Toolchains` 에 들어가면 자동으로 clion이 컴파일러를 잡는다.
- Apply를 눌러준다.

---

## 프로젝트 리로딩

![image](https://user-images.githubusercontent.com/43930419/111024824-11f6ad00-8424-11eb-9b5b-c79205f51dd2.png)

- 이전에 만들어둔 프로젝트는 run이 비활성화 되어있고 컴파일이 불가능한 경우가 있다.
- 새로운 컴파일러를 잡도록 Reload를 해준다.

---

## 결과

![image](https://user-images.githubusercontent.com/43930419/111058397-c8649b80-84d1-11eb-8e75-c876937ca819.png)


- c, c++ 개발세팅 끝!