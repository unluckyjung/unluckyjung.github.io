---
title: MAC CLI 환경에서 copy & paste 쉽게하기
date: 2023-08-19-14:00
categories:
- Tool

tags:
- MAC

---

## MAC cli 환경에서 copy & paste 쉽게하기
> 파일에 있는 내용을 명령어를 통해 클립보드에 넣고 뺴고 하는법을 알아봅니다.

---

## Goal
- `pbcopy`, `pbpaste` **맥북** 명령어를 통해 파일 내용을 복사 붙여넣기 하는방법을 알아봅니다.

---

## 복사
> pbpaste

```console
cat log.txt | pbcopy
```

- 위 명령어를 넣으면 `log.txt` 내용을 클립보드로 복사하게 됩니다.
- 단순히 cat 으로 출력된것을 마우스 스크롤을 통해서 복붙하면, 종종 줄바꿈이 들어가거나 해서 복사가 그대로 안되는것을 방지할 수 있습니다.

```console
echo "copy test" | pbcopy
```

- 위 명령어로는 echo 의 결과를 클립보드에 저장할 수 있습니다.

## 붙여넣기

```console
pbpaste
```

- 클립보드에 있는것을 콘솔창에 출력할수 있습니다.

```console
pbpaste > log2.txt
```

- `> 파일명` 을 붙여주면, 붙여넣기 하면서 파일이 생성되게 됩니다.


## 참고
- 예시로는 `.txt` 확장자를 들긴했지만 다른 확장자의 파일도 가능합니다. 
- 저의 경우에는 최근 RSA 기반의 공개키를 복사하는 상황에서 사용 했었습니다.
- 만약 linux 환경에서도 위 명령어를 사용하고 싶으시다면 [xclip 패키지](http://guileschool.com/2018/12/01/Use-the-pbcopy-and-pbpaste-of-Mac-on-Linux/)를 설치한뒤 사용할 수 있습니다.
  

---

## Conclusion
- mac 환경에서 `pbcopy`, `pbpaste` 를 이용하여 cli 환경에서 copy & paste 를 실수 없게 할 수 있다.

---

## Reference
- http://guileschool.com/2018/12/01/Use-the-pbcopy-and-pbpaste-of-Mac-on-Linux/

