---
title: Github Container Registry
date: 2022-04-07-20:00
categories:
- Github

tags:
- Git
- Github
- Docker


---

## 깃허브 컨테이너 레지스트리
> Github에서 (도커)이미지를 관리할 수 있습니다.

## Goal
- 깃허브 컨테이너 레스트리가 무엇인지 알아봅니다.
- 기존 Docker Hub 대비 장단점을 알아봅니다.

---

## What
> 깃허브 레포에서 (도커)이미지를 관리할 수 있는 기술

- 도커 허브처럼 원격 저장소인 깃허브 레포지토리에 도커 이미지를 같이 올려서 관리할 수 있습니다.
- 기존의 도커 이미지를 원격저장소에 올리고, 스케일 아웃같은 작업을 진행할때는 보통 [도커 허브](https://hub.docker.com/)를 이용 했을것입니다.
- 하지만 도커 허브의 경우에는 **6개월**간 이미지를 사용하지 않으면 이미지가 자동으로 **삭제되는 문제**가 있고, 아무래도 소스는 깃허브에서 관리하는데 컨테이너는 도커 허브에 따로 관리하니 **관리포인트가 늘어나는 단점**이 있습니다.


---

## 장점
> 깃허브에서 관리를 할 수 있습니다.

- 관리포인트가 줄어들게 됩니다.
- `README`를 넣어서 해당 이미지에 대한 설명을 추가할 수 있습니다.
- 접근 권한을 일부 저장소 조직으로 제한을 두어 관리할 수 있습니다.

![image](https://user-images.githubusercontent.com/43930419/162189776-8461af63-df86-47aa-b7cc-0eeeb3b4b63f.png)

- [공식문서 내용](https://docs.github.com/en/packages/working-with-a-github-packages-registry/migrating-to-the-container-registry-from-the-docker-registry#key-differences-between-the-container-registry-and-the-docker-registry) 에 다음과 같은 차이점들도 나열되어 있습니다.


---
  
## vs pakage registry
> 레포지토리에 묶여있느냐, 조직이나 계정에 속해 있느냐

- **pakage registry** 의 경우에는 레포지토리에 묶여있어서, 깃허브 저장소의 하위 기능이라고 볼 수 있습니다.
- 하지만 **container registry** 의 경우에는 개인 계정이나 조직에 속해 있어, 레포지토리에 무관하게 이미지를 업로드 할 수도 있습니다.

```
// pakage registry
docker.pkg.github.com/<GITHUB_ID>/<REPOSITORY_NAME>/<IMAGE_NAME>:<TAG>

// container registry
ghcr.io/<GITHUB_ID>/<IMAGE_NAME>:<TAG>
```

- **container registry** 의 경우에는 레포지토리 네임이 없어도 푸쉬가 되는것을 확인할 수 있습니다.


---

## Conclusion
- **Github Container Registry** 는 이미지를 깃허브에서 다룰 수 있게 된다.
- 이덕에 관리포인트가 줄어들고 권한설정이 용이해지는 장점을 가지고 있다.



---

## Reference
- https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry
- https://stackoverflow.com/questions/66223428/what-is-the-difference-between-github-container-registry-and-github-packages-for
- https://www.44bits.io/ko/post/news--github-container-registry-beta-release
- https://dev.to/github/github-container-registry-better-than-docker-hub-1o9k