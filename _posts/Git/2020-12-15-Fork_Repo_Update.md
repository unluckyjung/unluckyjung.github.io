---
title: Fork 해온 레포지토리 최신화 하기
date: 2020-12-15-23:00
categories:
- Git

tags:
- Git
- Github

---

## Fork 해온 레포지토리를 원본 레포지토리에 맞춰서 업데이트 해봅니다.
> remote 공간을 이용합니다.

---

## 원본 레포지토리를 remote 저장소에 추가

```console
git remote -v 
```
* 현재 로컬의 리모트 공간확인
    * 아마 `orgin`만 있을것.

<br>

```console
git remote add upstream {원본 레포지토리 주소}
```

* 원본 레포지토리를 upstream이라는 이름을 붙여서 remote 공간에 추가하기
    * upstream은 관례로 쓰이는 용어. 다른 이름을 붙여도 상관없다.

<br>

```console
git remote -v
```
* 현재 로컬의 리모트 공간확인
    * `orgin`과 `upstream` 둘다 있을것
   
<br>


```console
git fetch upstream

혹은

git fetch upstream branchName
```

* remote 에 있는 upstream을 가져옴.
    * 특정 브랜치만 가져오고 싶다면 브랜치 이름을 뒤에 붙여줌.

<br>


---

## 동기화

```console
git checkout master
```

* 기존에 가지고 있던 마스터 브랜치로 이동한다.

<br>

```console
git merge upstream/master
```

* 원본 저장소에서 내려받은 최신화 upstream 을 master에 머지한다.

<br>

```console
git push
```

* 지금까지 일어난 모든 작업은 **로컬환경** 에서 일어난 일이므로, push를 해준다.

<br>

---

## 내려받음과 동시에 동기화

```console
// upstream을 내려받는 단계에서

git fetch upstream

// 대신

git pull upstream 
```

* 만약 다음과 같이 fetch 대신 pull을 한다면
    * 동기화 과정에서 있었던 `merge` 과정을 생략할 수 있다.


---

## reabse를 이용하는 방법.

- 기본적으로 위의 과정과 거의 동일하다.

```
git checkout 로컬브랜치이름
git remote add upstream 원본저장소주소
git remote -v
git fetch upstream 원하는브랜치이름
git rebase upstream/로컬브랜치이름
```

---

## Reference
* https://velog.io/@k904808/Fork-%ED%95%9C-Repository-%EC%97%85%EB%8D%B0%EC%9D%B4%ED%8A%B8-%ED%95%98%EA%B8%B0
* http://taewan.kim/post/updating_fork/