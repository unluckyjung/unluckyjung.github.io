---
title: clion에서 freopen 사용하기 [File I/O]
date: 2021-03-14-15:10
categories:
- Tool

tags:
- IDE
- Clion
- Cpp

---

## clion에서 freopen 을 사용할 수 있도록 설정해봅니다.
> 정확히는 freopen만이 아닌 File I/O 를 할수 있게 설정해봅니다.


### Goal
- clion에서 freopen을 원하는 경로에서 할 수 있도록 해본다.

---

## freopen
> ps를 할때 자주쓰는 파일을 읽어오는 함수. 삼성 역량테스트에서는 아예 템플릿으로 제공해준다.

```c++
freopen("input.txt", "r", stdin);
```

- vs와 다르게 clion의 경우 소스코드와 같은 위치에 두어도 freopen으로 읽지 못한다.
- 내 기억에 같은 jetbrain에서 만든 다른 ide인 `intellij`의 경우에도, 상대경로가 안먹어서 절대 경로로 파일 경로를 준 기억이 난다.
  - 이부분은 나중에 확인하고 포스팅에 추가할 예정.

---

## 읽어올 파일 생성

![image](https://user-images.githubusercontent.com/43930419/111057836-b41e9f80-84cd-11eb-84e0-cbb706f92de1.png)

- 디렉토리를 만들고, freopen으로 읽을 파일을 넣어준다.
  - 나의 경우 `input_data > input.txt` 로 설정해두었다.
  - **필자처럼 꼭 디렉토리를 만들어줄 필요는없다!** 이유는 뒤에서 설명한다.

---

## 경로 설정
![image](https://user-images.githubusercontent.com/43930419/111057831-ad902800-84cd-11eb-952b-1c3334e5ef9a.png)

- `Edit Configurations` 에 들어가준다.

![image](https://user-images.githubusercontent.com/43930419/111057846-be409e00-84cd-11eb-8ce9-79ced563d935.png)

- `Working directory` 를 보면 빈 공백인것을 확인할 수 있다.
- 즉, 소스파일이 위치한 경로가 디폴트로 잡히고 있지 않다는것을 확인할 수 있다.
  - vs의 경우 디폴트로 잡힌다
- 이로 인해서 freopen으로 읽어오지 못했던것이다.
- 우측끝의 폴더 모양을 클릭 해주자.

![image](https://user-images.githubusercontent.com/43930419/111057849-c8629c80-84cd-11eb-8405-044d5d091337.png)
- 읽어올 파일이 있는 경로를 설정해준다.
- 위에서 **필자처럼 꼭 디렉토리를 만들어줄 필요는없다!** 라고 말했었다.
- 디렉토리를 만들지 않고 `main.cpp` 가 위치한 곳에 `input.txt` 를 만들어주었다면, main과 input.txt 를 같이 가지고 있는 디렉토리를 경로로 잡으면 된다.
- 필자의 경우 `input_data` 안에 `input.txt`가 있으므로, `input_data`까지를 경로로 잡겠다.


![image](https://user-images.githubusercontent.com/43930419/111057855-d1536e00-84cd-11eb-9274-8acb3a06f59d.png)

---

## 결과 확인

![image](https://user-images.githubusercontent.com/43930419/111059186-d5d05480-84d6-11eb-9a3b-963d58f91415.png)


- 이제 input_data 를 기준으로 상대 경로로 접근할 수 도 있다.
- 예를들어 input_data 안에 input_data2 라는 디렉토리가 있고, 그안의 파일을 읽어오고 싶다면

```
freopen("./input_data2/input.txt", "r", stdin);
```
와 같은 방법으로도 접근이 가능해진다.

---

## Reference
- https://blog.naver.com/strike0115/221519898666