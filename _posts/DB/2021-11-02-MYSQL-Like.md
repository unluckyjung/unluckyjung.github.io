---
title: Mysql Like 동작 방식
date: 2021-11-02-18:00
categories:
- DB

tags:
- MYSQL
- DB
- DataBase

photos:
- /post_images/mysql.png

---

# Mysql의 Like는 어떻게 작동할까
> Turbo Boyer-Moore 알고리즘을 이용하고 있습니다.

---

## Goal
- Mysql 의 Like는 어떤 자료구조, 알고리즘을 통해서 매칭되는 문자열을 찾는지 알아봅니다.

---

## 개요
> 팀프로젝트의 자동완성 기능

- Babble 프로젝트에서 태그명, 게임명 자동완성 기능을 제공하고 있습니다.
- 현재 내부적인 구현은 입력때 마다 request를 보내고, 해당 문자열에 해당하는 객체를 Like를 이용한 쿼리를 통해서 응답하고 있습니다. [Github 링크](https://github.com/woowacourse-teams/2021-babble/blob/develop/back/babble/src/main/java/gg/babble/babble/domain/repository/TagRepository.java)
- 하지만 문자열 자동완성의 경우에는 Trie 자료구조를 이용하여, 매칭되는 문자열을 찾는것이 좋다는것을 알고 있는 입장에서, 매번 요청때마다 Like 처리를 통해 응답하는것을 비효율적이라고 생각했습니다.
- 여기서 이어지는 궁금증으로 **실제로 MYSQL 에서는 Like에 대한 처리를 어떻게 하는지 궁금해져서 찾아보았습니다.**

---

## Mysql에서는 Turbo Boyer-Moore를 이용합니다.

> If you use ... LIKE '%string%' and string is longer than three characters, MySQL uses the Turbo Boyer-Moore algorithm to initialize the pattern for the string and then uses this pattern to perform the search more quickly.

- [공식 문서](https://dev.mysql.com/doc/refman/8.0/en/index-btree-hash.html) 에 적혀 있는 내용이다.

- 일반적으로 인덱스가 걸려있는 경우에는 B-Tree 를 이용하고 있고, 그렇지 않은 경우에는 [Turbo Boyer-Moore](http://www-igm.univ-mlv.fr/~lecroq/string/node15.html) 알고리즘을 통해서 매칭되는 문자열을 찾는다고 기술되어 있습니다.


---

## RDBMS에서는 Trie나 아호코라식을 사용하지 않습니다.

```
Tries and Aho-Corasick aren't designed to deal with the storage hierarchy of pages that define most SQL databases (even column stores). Btrees give fast access to the right page, then typically the page is scanned.

The costs of traversing the btree index and the final scan are not large, and don't typically present as major parts of the query cost.

I'm open to suggestions how a trie could be set up as an index in a higher level of the storage hierarchy, maybe as an implementation of the btree index. That seems tractable, but possibly of very marginal gain. Queue research project.

I don't see immediately where an Aho-Corasick machine would be generated or stored in the context of a frequently updated dictionary of indexed strings in the storage hierarchy. Sounds like a deeper research problem.
```

[출처](https://www.quora.com/Why-dont-SQL-DBs-use-Trie-or-Aho-Corasick-automation-to-index-strin)  

- KMP의 경우에는, 보이어 무어와 굉장히 비슷한 성능을 가지나, 긴 문자열에 대한 처리에서는 보이어무어보다 불리하기 때문에 사용되지 않는것으로 파악됩니다.
- 그 외, 아호코라식, Trie, 라빈 카프 알고리즘의 경우에도 RDBMS에서는 효율적이지 않다고 파악된것 같습니다.
  - 해당 이유를 좀더 고민해보고 싶었는데, 너무 야크쉐이빙이되는것 같아서 일단은 넘어가려고 합니다.. `2021.10.31`
  - 나중에 여유가 있을때 좀더 자료를 찾아보며 고민해보려고 합니다.

### 참고
- https://www.quora.com/Why-dont-SQL-DBs-use-Trie-or-Aho-Corasick-automation-to-index-strin
- https://iq.opengenus.org/kmp-vs-boyer-moore-algorithm/


---

## Babble 자동완성에 대한 고민
> 매번 DB와 커넥션 하면서 보이어 무어를 이용한 패턴매칭 vs 메모리상에서 Trie로 관리하면서 자동완성

- 현재 구현된 방식은 전자입니다.
- 하지만 DB와 커넥션을 만들고 끊는 과정은 많은 자원을 소모하기 때문에, 후자의 경우가 성능에 대해서 유리하다고 생각됩니다.
  - 또 한편으로는 자동완성 기능에 대한 기대 퍼포먼스값은 유저 입장에서 관대하기 때문에(1초 정도 기다려도 큰 거부감을 느끼지 않습니다.) 그냥 전자로 구현해도 괜찮이 않을까 싶다는 생각이들었습니다.

- 이 부분은 나중에 Trie를 사용해 리팩토링해보고, 성능 측정을 통해 비교 해볼까 합니다.

---

## Reference

- https://dev.mysql.com/doc/refman/8.0/en/index-btree-hash.html
- https://www.quora.com/Why-dont-SQL-DBs-use-Trie-or-Aho-Corasick-automation-to-index-strin
- https://iq.opengenus.org/kmp-vs-boyer-moore-algorithm/

