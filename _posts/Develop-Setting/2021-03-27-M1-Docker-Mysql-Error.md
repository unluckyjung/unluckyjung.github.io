---
title: no matching manifest for linux/arm64/v8 in the manifest list entries 해결
date: 2021-03-27-19:56
categories:
- Develop-Setting

tags:
- Docker
- M1
- MAC
- MYSQL

---

## M1 버전의 docker를 사용시 발생하는 에러를 해결해봅니다.
> ERROR: no matching manifest for linux/arm64/v8 in the manifest list entries 에러 해결

### goal
- m1 (silicon) 버전 docker에 mysql 설치시 발생하는 문제를 해결해봅니다.

---

## 오류가 발생 하는 상황

```
version: '3'
services:
  local-db:
    image: library/mysql:5.7
    container_name: local-db
    restart: always
    ports:
      - 13306:3306
    environment:
      MYSQL_ROOT_PASSWORD: secret
      TZ: Asia/Seoul
    volumes:
      - ./db/mysql/data:/var/lib/mysql
      - ./db/mysql/init:/docker-entrypoint-initdb.
```

- 도커 설정을 위해 `docekr-compose.yml` 을 위와 같이 만든뒤

```
docker-compose up -d
```

- compose 명령을 통해 도커를 실행시키면 아래와 같은 오류를 만나게 됩니다.
  
```
ERROR: no matching manifest for linux/arm64/v8 in the manifest list entries
```
- 에러메시지를 그대로 읽어보면, `linux/arm64/v8` 에 해당하는 manifest가 없다고 합니다.

### yml 파일이 뭔가요?
> 컨테이너를 실행, 관리할 수 있는 설정 파일

- 도커 컨테이너에 필요한 옵션, 의존성 설정파일을 이곳에 적어두면 컴포즈용 명령어를 통해 컨테이너들을 실행하거나 관리할 수 있습니다. 
  - 자세한 설명이 있는 [공식사이트 링크](https://docs.docker.com/compose/)


---

## 오류 해결
> platform을 명시해 줍니다.

```
version: '3'
services:
  local-db:
    platform: linux/x86_64    # 추가된 라인
    image: library/mysql:5.7
    container_name: local-db
    restart: always
    ports:
      - 13306:3306
    environment:
      MYSQL_ROOT_PASSWORD: secret
      TZ: Asia/Seoul
    volumes:
      - ./db/mysql/data:/var/lib/mysql
      - ./db/mysql/init:/docker-entrypoint-initdb.
```

- `yml` 파일을 다시 열고 **local-db** 아랫줄에
- `platform: linux/x86_64` 을 추가해주어(**4번째 라인**) 플랫폼을 명시 해줍니다.

![image](https://user-images.githubusercontent.com/43930419/112718079-0337ec00-8f34-11eb-9e83-a64a7deb948e.png)


```console
docker-compose up -d
```

- 실패했었던 compose 명령을 다시 쳐보면 오류가 해결된것을 확인할 수 있습니다.

---

## Reference
- https://stackoverflow.com/questions/65456814/docker-apple-silicon-m1-preview-mysql-no-matching-manifest-for-linux-arm64-v8
- https://www.44bits.io/ko/post/almost-perfect-development-environment-with-docker-and-docker-compose
- https://docs.docker.com/compose/
- https://cultivo-hy.github.io/docker/image/usage/2019/03/14/Docker%EC%A0%95%EB%A6%AC/