---
title: 이름, 경로가 변경된 파일의 이전 commit history를 추척할 수 있을까?
date: 2020-12-23-01:00
categories:
- Git

tags:
- Git


---

## 파일 위치를 이동하거나, 이름 변경시에도 git commit 히스토리를 유지할 수 있을까?
> git에서는 가능하다. 그러나 github에서 불가능하다.

* 파일 이동을 하는 커밋 로그의 컨벤션을 무엇으로 해야할까 고민중
    * 트래킹중인 파일의 이동, 이름 변경을 할경우 이전의 로그가 다 추적이 안되는데, 그것을 어떻게 보관하는 방법이 없을까? 하는 의문점이 생겼다.
    * 파일 이동시 커밋 헤더는 `refactor` 로 하기로 했다.

---

## 일단, 원하는 내가 방향 으로는 [불가능] 하다
> git 자체만으론 편법으로 가능은 하나, 안하는것이 좋다.

* 결론만 이야기하면 **불가능** 하다고 한다.
    * 왜냐하면 사실 git은 `file`이나 `움직임`으로 트래킹 하지 않기 때문...
    * git은 `content` 로 트래킹 하기 때문이다.

* `-follow` 을 이용하면  **git** 에서는 가능하다고 하나 [성능상의 이슈](https://github.com/gogs/gogs/issues/2487)가 있다.
    *  그리고 `github` 에는 표시가 안되므로, 생각해보면 크게 유용한거 같지도 않다.
* 솔직히 기능을 지원헀으면 하는데, 그러면 git의 동작방식과 구조를 다 뜯어 고쳐야 하지 않을까 라는 생각이 든다.

---

## Reference
* https://stackoverflow.com/questions/2314652/is-it-possible-to-move-rename-files-in-git-and-maintain-their-history
* https://lumiloves.github.io/2018/06/25/how-to-preserve-file-history-when-renaming-or-moving-in-git
* https://git.wiki.kernel.org/index.php/GitFaq#Why_does_Git_not_.22track.22_renames.3F
* https://github.com/gogs/gogs/issues/2487
