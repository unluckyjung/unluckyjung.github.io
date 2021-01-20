---
title: M1 MAC에서 Jekyll 블로깅을 위한 로컬 환경 구축하기
date: 2021-01-20-23:56
categories:
- Develop-Setting

tags:
- Ruby
- Jekyll
- M1
- MAC

photos:
- https://jekyllrb-ko.github.io/img/logo-2x.png
---

## M1 MAC 에서 지킬 블로그 운영을 위한 로컬 환경을 구축해봅니다.
> 루비 설치부터, 로컬 구동까지. 디펜던시 지옥을 해결하며

---

## 루비 설치
> [전체 흐름 링크](https://jojoldu.tistory.com/288)


### m1 칩을 위한 사전세팅

```
xcode-select --install
```

* CLT 설치 `Command Line Tools`
* 굳이 `xcode` 를 설치할 필요가 없다.
* [참고 링크](https://atoz-develop.tistory.com/entry/%EB%A7%A5-CLTCommand-Line-Tools-%EC%84%A4%EC%B9%98-%EB%B0%A9%EB%B2%95Xcode-%EC%97%86%EC%9D%B4)

```
brew install gcc
```

* gcc 설치
* 기본적으로 깔려 있지만, 해당 작업을 생략할시 디펜던시 에러가 발생한다.

---

### 본격적인 루비 설치

```
brew install rbenv ruby-build
```
* rbenv 설치

```
rbenv versions

* system (set by /Users/yoonsung/.rbenv/version)
```
* 현재 시스템에서 사용중인 ruby 버전 확인하기
* MAC에 기본적으로 있는 ruby의 버전을 확인할 수 있다.
* `sudo`를 이용해 루트 권한으로 해당 ruby를 사용할 수 있으나, **권장하지 않는다**
* 그리고 필자가 사용중인 지킬 블로그는 루비 버전이 `2.6.0` 이하의 경우에만 호환되므로, 그 이하의 버전을 받아 설치할 것이다.

```
rbenv install -l    // 설치 가능한 버전보기
rbenv install -list-all     // 설치 가능한 모든 버전 보기
rbenv install {원하는 버전}
```
* 루비 설치하기

```
rbenv versions

* system (set by /Users/yoonsung/.rbenv/version)
  2.5.8
```
* 방금 설치한 2.5.8 이 추가된것을 확인 할 수 있다.

```
rbenv global 2.5.8

-- 버전 확인

rbenv versions

  system
* 2.5.8 (set by /Users/yoonsung/.rbenv/version)
```
* 2.5.8 버전으로 바꾸어 준다.

---

## 터미널 설정 바꾸어주기

```
vi ~/.zshrc
```
* zsh 를 사용하니, zsh 설정 파일을 열어준다.

```
export PATH={$Home}/.rbenv/bin:$PATH && \
eval "$(rbenv init -)"
```

* PATH 추가를 취해 위의 두줄을 추가해준다.

```
source ~/.zshrc
```
* 환경설정 바꾼것을 적용시켜준다.

---

## 지킬 블로그 구동을 위한 설정하기
> [이전에 쓴 글](https://unluckyjung.github.io/github/2020/02/16/Jekyll_start/) 를 참고했다.

```
gem install bundler
rbenv rehash    // !!!!!!!필수!!!!!!!!!!
```

* `rehash` 를 안해줘서 디펜던시 지옥에 빠졌었다.
    * `You must use Bundler 2 or greater with this lockfile`
        * https://github.com/jekyll/jekyll/issues/7463
    * 이것부터 시작하여 의존되는 에러를 하나씩 해결 해나갔지만, **리해싱** 전까지는 `bundler install` 은 계속 먹통이었다.

```
bundler install
bundle exec jekyll serve
```

* `localhost:4000` 로컬 환경에서 돌아가는것을 확인! 끝!
* 설명이 굉장히 많이 생략되었는데 [이전에 쓴 글](https://unluckyjung.github.io/github/2020/02/16/Jekyll_start/) 과 중복되는 내용이 많아 생략했다.

---

## Reference
* https://jojoldu.tistory.com/288
* https://atoz-develop.tistory.com/entry/%EB%A7%A5-CLTCommand-Line-Tools-%EC%84%A4%EC%B9%98-%EB%B0%A9%EB%B2%95Xcode-%EC%97%86%EC%9D%B4
* https://smartbase.tistory.com/43