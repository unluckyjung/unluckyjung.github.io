---
title: Intellij webClient plugin 사용법
date: 2022-03-08-18:00
categories:
- Tool

tags:
- IDE
- Intellij

---

## Intellij webClient plugin을 이용해 http requset 만들어보기
> postman을 사용하지 않고 request를 만들고, response를 확인해봅니다.

---

## Goal
- 인텔리제이에서 지원하는 `.http` 를 이용해 http 요청을 보내고 응답을 콘솔에서 확인해봅니다.
- POST시 json 파일도 요청에 넣어서 보내봅니다.

---

## 요청용 스크립트 파일 생성
![image](https://user-images.githubusercontent.com/43930419/154459356-9f7ffe87-229a-4025-ad0f-30663205ce2a.png)

- 우측을 클릭해서 Http Request를 만들어봅니다.

## 스크립트 내용 입력
![image](https://user-images.githubusercontent.com/43930419/154459395-102748f9-a90f-4443-91e2-6919b10b88ce.png)

![image](https://user-images.githubusercontent.com/43930419/154460357-f2aadc05-f655-4bfb-afbb-b1a38b8dda05.png)


- 위와같이 HTTP 메소드 이름과, 요청을 적어주고 **반드시 어플리케이션을 실행한뒤** 초록색 화살표를 눌러 주면 요청이 보내지고, 요청에 따른 응답을 받을 수 있게됩니다.
- Json 타입을 보내고 싶다면 `Content-Type` 역시 설정이 가능하고, 이외 다른 쿠키와 같은 값들도 넣어줄 수 있습니다. 



![image](https://user-images.githubusercontent.com/43930419/154459442-ceac1a98-f6c2-42f3-a8df-d714a6d5cd46.png)

- POST 단맨 아래의 `<.member.json` 의 경우 해당 json파일을 POST요청에 넣는다는것을 의미합니다.
- `member.json` 에는 올바른 json 타입의 정보를 적어주면 됩니다.


![image](https://user-images.githubusercontent.com/43930419/155847973-882bb2ab-42c3-4ef5-972b-4675dc4270aa.png)

- 파일로 따로 분리할 필요성이 느껴지지 않는다면, 작성한 요청 밑에 바로 적어주어도 요청생성이 가능합니다.


### 원클릭 Request 자동생성

![image](https://user-images.githubusercontent.com/43930419/155847399-29c1897f-bebd-425b-983a-1316efd07ce4.png)

![image](https://user-images.githubusercontent.com/43930419/155850537-1aa433a9-6939-4f87-9c56-1d3d8e8bf890.png)



- 컨트롤러단에서 `Open in HTTP` Client를 클릭해 요청메시지를 자동으로 생성하는 방법도 있습니다.


---

## Conclusion
- 기존 PostMan을 통해서 작업을 IntelliJ 에서 제공하는 **webClient plugin** 을 이용해 간편하게 HTTP 요청을 보내고 응답을 받아볼 수 있다.

---

## Reference
- https://www.jetbrains.com/help/idea/http-client-in-product-code-editor.html#converting-curl-requests
- https://jojoldu.tistory.com/266