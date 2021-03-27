---
title: M1 Docker, Workbench 설치
date: 2021-03-27-20:26
categories:
- Develop-Setting

tags:
- Docker
- M1
- MAC
- Tool
- WorkBench

---

## M1 맥북에 도커와 워크벤치를 설치해봅니다.
> 설치시 발생하는 오류도 해결해봅니다.

### Goal
- M1 맥북에 도커를 설치합니다.
- M1 맥북에 워크벤치를 설치합니다.

---

## Docker 설치
> **반드시** Preview 버전으로 설치해야 합니다. 

- [링크](https://docs.docker.com/docker-for-mac/apple-m1/) 로 가서 받아줍니다.

<br>


![image](https://user-images.githubusercontent.com/43930419/112717039-4fcbf900-8f2d-11eb-80c8-c70f9c960942.png)
![image](https://user-images.githubusercontent.com/43930419/112717041-55294380-8f2d-11eb-8251-eff14f601e9b.png)

- 구글에 docker mac install 검색 후, 다음 링크로 그냥 들어가서 다운로드 하는 경우가 있으신데
- :x: **위 사진에 나온, 두개의 링크에선 받으면 안됩니다. apple silicon 버전이 안보이니 위화감을 느끼셔야 합니다** :x:
- m1 slicon 버전 다운로드 위치를 상대적으로 찾기 힘들게 해놨던데 왜 그런건진 모르겠습니다.

---

## workbench 다운
> workbench는 m1버전이 따로 배포가 되고 있지 않습니다.

- 최신 버전을 설치하면 오류가 납니다. `2021-03-27 기준 8.0.23`
- 한단계 아래 버전인 [8.0.22 버전](https://downloads.mysql.com/archives/workbench/)을 받아 줍니다.

![image](https://user-images.githubusercontent.com/43930419/112717155-1a73db00-8f2e-11eb-83e9-38967b4cd4d6.png)

- 이 문제는 `m1` 이여서가 아니라 `빅서` 문제인것으로 추측됩니다.
- [스택 오버플로우 링크](https://stackoverflow.com/questions/65792436/mysql-workbench-quit-unexpectedly-on-mac-os-big-sur-11-1)
- **22버전**도 에러가 나는 경우도 있다하니 아예 속편하게 **17버전**까지 내려서 받는것도 하나의 방법이겠습니다.


## workbench 설치
> 악성 소프트웨어가 있는지 확인할 수 없기 때문에 열 수 없습니다. 

- 위와 같은 **메시지**가 뜨면서 설치가 거부되는 경우가 존재합니다.
- 아래 순서를 따라 해결합니다.
  - 시스템 환경설정
  - 보안 및 개인정보
  - 일반 탭 > 확인없이 열기 클릭

- [참고한 링크](https://seyul.tistory.com/78)에 이미지와 함께 자세히 설명되어 있습니다.

---

## ETC
> Docker yml에 mysql 설정시 발생하는 오류 해결

- 도커를 compose 명령어를 통해 설정, 실행할경우
- `ERROR: no matching manifest for linux/arm64/v8 in the manifest list entries` 라는 에러를 만날 수 도 있습니다.

- [해당 문서](https://unluckyjung.github.io/develop-setting/2021/03/27/M1-Docker-Mysql-Error/)를 보고 해결합니다.

---

## Reference

### 도커
- https://itkoo.tistory.com/10
- https://stackoverflow.com/questions/65456814/docker-apple-silicon-m1-preview-mysql-no-matching-manifest-for-linux-arm64-v8

### 워크벤치
- https://stackoverflow.com/questions/65792436/mysql-workbench-quit-unexpectedly-on-mac-os-big-sur-11-1
- https://seyul.tistory.com/78
