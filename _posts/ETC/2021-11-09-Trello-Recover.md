---
title: Trello 카드 내용 복구하기
date: 2021-11-09-17:30
categories: 
- ETC

tags:
- Trello

---

## Trello에서 실수로 내용을 날려 먹었을때 복구해봅니다.
> 카드의 내용(description)을 복구해봅니다.

---

## 개요

- Trello의 card를 수정중 실수로 잘못된 키를 한번 누르고, 카드 밖을 클릭하여 이전의 모든 description이 날아가고 잘못입력 된 데이터만 카드에 남은 사태가 발생 했습니다.
- 복구를 위해서 Activity 내역을 살펴 보았지만, 이전카드에 적힌 내용 이력을 찾아볼순 없었습니다.
- **"trello recover description"** 라는 키워드로 검색해 보았고, 복구 방법을 찾을 수 있었습니다.  


> Recovering the description or title of a card that has been changed  
> If someone changes the description of a card, it may appear that the entire description has been deleted.  
> However, Trello tracks these changes in the background  


- https://help.trello.com/article/783-recovering-the-description-of-a-card-that-has-been-changed
- [링크](https://help.trello.com/article/783-recovering-the-description-of-a-card-that-has-been-changed)에 모든 것이 설명되어있지만, 간략하게 설명해드리겠습니다.

---

## 복구 과정

### Step1
> description을 날려먹은 카드의 ID를 찾습니다.

- 카드의 아이디는 해당 카드를 클릭했을때 나오는 url에서 찾을 수 있습니다.

```
https://trello.com/c/{CARD-ID}/814-multiple-log-in-credentials

https://trello.com/c/hpAcP7IS/814-multiple-log-in-credentials
```

- 이 주소에서 카드의 ID는 `hpAcP7IS`가 됩니다.

​

### Step2
> 크롬을 사용한다는 가정하에 , [확장 프로그램](https://chrome.google.com/webstore/detail/jsonview/chklaanhfefbnpoihckbnefhakgolnmc?hl=ko)을 다운받습니다.

- 크롬에서 json 호출시 비정렬된 json이 넘어오는데, 그걸 가독성 좋게 포매팅 해주어 내용을 쉽게 확인할 수 있게 해줍니다.

​
### STEP3 
> 카드 업데이트 이력을 확인합니다.

- 아래와 같은 형식으로 주소창에 입력합니다.

```
https://trello.com/1/cards/[CARD-ID]/actions?filter=updateCard:desc

// card Id = hpAcP7IS
https://trello.com/1/cards/hpAcP7IS/actions?filter=updateCard:desc
```

- 이때 [card ID]부분에 1에서 찾은 카드의 ID를 입력합니다. `(hpAcP7IS)`


- 여기까지 잘 따라왔다면 지금까지 해당 카드의 변경된 모든 이력을 확인 할 수 있게됩니다.
- `data.old.desc` 에서.. 날려먹기 이전의 description을 찾을 수 있게됩니다.


---

## Reference
- 과거 [네이버 블로그](https://blog.naver.com/ybook2006/221760169654)에 적었던 글을 이전했습니다.
- https://help.trello.com/article/783-recovering-the-description-of-a-card-that-has-been-changed