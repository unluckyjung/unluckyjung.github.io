---
title: Spring WebSocket 구현-1
date: 2021-08-19-15:00
categories:
- Spring

tags:
- Spring
- WebSocket
- Web

---

## Spring WebSocket qucik Start-1
> 스프링을 이용해 웹소켓 서버를 구현해봅니다.

## Goal
- 웹소켓에 대한 개념 설명보다는 **구현** 자체에 집중해봅니다.
- [공식문서 가이드](https://spring.io/guides/gs/messaging-stomp-websocket/)를 매우 참고하여, 한개의 방에서 입장, 퇴장 메시지를 전송하는 app을 구현해봅니다.

---

## 환경
- IntelliJ 2021.2
- java 8
- gradle 7.1.1


---

## 초기설정
- https://start.spring.io/ 에 들어갑니다.

![image](https://user-images.githubusercontent.com/43930419/129760758-1a140bb0-3186-4d04-a989-06539f7ce270.png)

- 위와 같이 선택한뒤 내려받습니다.


### 의존성 추가

```java
implementation 'org.webjars:webjars-locator-core'
implementation 'org.webjars:sockjs-client:1.0.2'
implementation 'org.webjars:stomp-websocket:2.3.3'
implementation 'org.webjars:bootstrap:3.3.7'
implementation 'org.webjars:jquery:3.1.1-1'
```

- `build.gradle` 에 의존성들을 추가해줍니다.

<br>

```java
plugins {
	id 'org.springframework.boot' version '2.5.3'
	id 'io.spring.dependency-management' version '1.0.11.RELEASE'
	id 'java'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '1.8'

repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-websocket'

	implementation 'org.webjars:webjars-locator-core'
	implementation 'org.webjars:sockjs-client:1.0.2'
	implementation 'org.webjars:stomp-websocket:2.3.3'
	implementation 'org.webjars:bootstrap:3.3.7'
	implementation 'org.webjars:jquery:3.1.1-1'

	testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

test {
	useJUnitPlatform()
}
```

- `build.gradle` 파일의 형태는 최종적으로 위와 같은 형태가 됩니다.

---

## 클라이언트와 서버가 입장, 퇴장 메시지를 주고받을 DTO를 만듭니다.

```java
public class EnterRequest {

    private String userName;

    public EnterRequest() {
    }

    public EnterRequest(final String userName) {
        this.userName = userName;
    }

    public String getUserName() {
        return userName;
    }
}

```

<br>

```java
public class EnterResponse {

    private String userName;

    public EnterResponse() {
    }

    public EnterResponse of(final EnterRequest enterRequest) {
        return new EnterResponse(enterRequest.getUserName());
    }

    public EnterResponse(final String userName) {
        this.userName = userName;
    }

    public String getUserName() {
        return userName;
    }
}
```

- 누가 입장했는지를 표시해주기 위한, 간단한 메시지 DTO를 만듭니다.


---

## 화면에 그려줄 클라이언트 코드

```js
var stompClient = null;

function setConnected(connected) {
  $("#connect").prop("disabled", connected);
  $("#disconnect").prop("disabled", !connected);
  if (connected) {
    $("#conversation").show();
  }
  else {
    $("#conversation").hide();
  }
  $("#join").html("");
}

function connect() {
  var socket = new SockJS('/ws');
  stompClient = Stomp.over(socket);
  stompClient.connect({}, function (frame) {
    setConnected(true);
    console.log('Connected: ' + frame);
    stompClient.subscribe('/subscribe/join', function (join) {
      showJoinMessage(JSON.parse(join.body).userName + "님이 입장하셨습니다.");
    });
  });
}

function disconnect() {
  if (stompClient !== null) {
    stompClient.disconnect();
  }
  setConnected(false);
  console.log("Disconnected");
}

function sendName() {
  stompClient.send("/app/join", {}, JSON.stringify({'userName': $("#name").val()}));
}

function showJoinMessage(message) {
  $("#join").append("<tr><td>" + message + "</td></tr>");
}

$(function () {
  $("form").on('submit', function (e) {
    e.preventDefault();
  });
  $( "#connect" ).click(function() { connect(); });
  $( "#disconnect" ).click(function() { disconnect(); });
  $( "#send" ).click(function() { sendName(); });
});
```

<br>

```html
<!DOCTYPE html>
<html>
<head>
  <title>Hello WebSocket</title>
  <link href="/webjars/bootstrap/css/bootstrap.min.css" rel="stylesheet">
  <link href="/main.css" rel="stylesheet">
  <script src="/webjars/jquery/jquery.min.js"></script>
  <script src="/webjars/sockjs-client/sockjs.min.js"></script>
  <script src="/webjars/stomp-websocket/stomp.min.js"></script>
  <script src="/app.js" charset="UTF-8"></script>
</head>
<body>
<noscript><h2 style="color: #ff0000">Seems your browser doesn't support Javascript! Websocket relies on Javascript being
  enabled. Please enable
  Javascript and reload this page!</h2></noscript>
<div id="main-content" class="container">
  <div class="row">
    <div class="col-md-6">
      <form class="form-inline">
        <div class="form-group">
          <label for="connect">WebSocket connection:</label>
          <button id="connect" class="btn btn-default" type="submit">Connect</button>
          <button id="disconnect" class="btn btn-default" type="submit" disabled="disabled">Disconnect
          </button>
        </div>
      </form>
    </div>
    <div class="col-md-6">
      <form class="form-inline">
        <div class="form-group">
          <label for="name">What is your name?</label>
          <input type="text" id="name" class="form-control" placeholder="Your name here...">
        </div>
        <button id="send" class="btn btn-default" type="submit">Send</button>
      </form>
    </div>
  </div>
  <div class="row">
    <div class="col-md-12">
      <table id="conversation" class="table table-striped">
        <thead>
        <tr>
          <th>Chatting Room</th>
        </tr>
        </thead>
        <tbody id="join">
        </tbody>
      </table>
    </div>
  </div>
</div>
</body>
</html>
```

<br>

```css
body {
  background-color: #f5f5f5;
}

#main-content {
  max-width: 940px;
  padding: 2em 3em;
  margin: 0 auto 20px;
  background-color: #fff;
  border: 1px solid #e5e5e5;
  -webkit-border-radius: 5px;
  -moz-border-radius: 5px;
  border-radius: 5px;
}
```

- 공식문서의 코드와 거의 동일합니다.

---

## Server Code

### Controller
> 메시지를 받고, 전송하는 역할을 수행

```java
import com.example.simplewebsocket.dto.JoinRequest;
import com.example.simplewebsocket.dto.JoinResponse;
import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.messaging.handler.annotation.SendTo;
import org.springframework.stereotype.Controller;

@Controller
public class JoinMessageController {

    @MessageMapping("/join")
    @SendTo("/subscribe/join")
    public JoinResponse joinMessage(final JoinRequest joinRequest) {
        return JoinResponse.of(joinRequest);
    }
}
```

<br>

```java
@MessageMapping("/join")
```

- `@GetMapping` 처럼 **/join** 이라는 url를 해당 메소드로 매핑시킵니다.


<br>

```java
@SendTo("subscribe/join")
```

- `subscribe/join` 을 구독하고 있는 모든 클라이언트에게 리턴되는 값을 브로드 캐스팅 합니다.


```js
// Client
stompClient.subscribe('/subscribe/join', function (join) {
    showJoinMessage(JSON.parse(join.body).userName + "님이 입장하셨습니다.");
});
```

- 위와 같이 `subscribe/join` 를 구독하고 있는 클라이언트들에게 메시지를 전부 보낼 수 있게 됩니다.


---

### Config
> 스프링단에서 `WebSocket` 을 활성화 하는 작업을 수행해 보겠습니다.

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.simp.config.MessageBrokerRegistry;
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;
import org.springframework.web.socket.config.annotation.WebSocketMessageBrokerConfigurer;

@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(final StompEndpointRegistry registry) {
        registry.addEndpoint("/ws").withSockJS();
    }

    @Override
    public void configureMessageBroker(final MessageBrokerRegistry config) {
        config.enableSimpleBroker("/subscribe");
        config.setApplicationDestinationPrefixes("/app");
    }
}
```


<br>


```js
// client
function connect() {
  var socket = new SockJS('/ws');
  ...
}
```

```java
// server
public void registerStompEndpoints(final StompEndpointRegistry registry) {
    registry.addEndpoint("/ws").withSockJS();
}
```

- 엔드포인트가 어딘지를 설정하여, 최초 커넥션이 어디쪽으로 이루어져야 하는지를 설정해줍니다.
- 클라이언트 입장에서, `/ws`로 접근할시, 웹소켓 커넥션을 만들게 됩니다.
  - 뒤의 `withSockJS()` 은 fallback 옵션입니다. 
  - 웹소켓을 사용하지 못하는 경우에 대체 전송방법을 찾아 전송을 수행할 수 있도록 해줍니다.

<br>


```java
@Override
public void configureMessageBroker(final MessageBrokerRegistry config) {
    config.enableSimpleBroker("/subscribe");
    config.setApplicationDestinationPrefixes("/app");
}
```

<br>

코드 설명에 앞서, 브로커라는 존재에 대해서 간단히 설명하고 지나가겠습니다.  
브로커는 말 그대로 **중개자**의 역할을 수행한다고 생각하시면 편합니다.

```
A가 1번방에 연결
B가 1번방에 연결
C가 1번방에 연결

이 상황일때
A가 1번방에 메시지를 전송한다면,

1번방을 중개하는 브로커가 그 메시지를 받아서, 1번방 구독자들에게 메시지를 전부 전송(브로드캐스팅) 하게 됩니다.
A (메시지)-> 1번방 브로커 -> 1번방 유저들 {A,B,C}
```

---

> enableSimpleBroker

```java
@Override
public void configureMessageBroker(final MessageBrokerRegistry config) {
    config.enableSimpleBroker("/subscribe");
    ...
}
```

- `enableSimpleBroker(/url)` 
- url에 해당하는 경로에 **SimpleBroker**룰 등록합니다.
  - **/url** 접두사가 붙은 메시지가 있다면, 브로커는 해당 url을 구독한 클라이언트들에게 메시지를 전송(브로드캐스팅) 하게 됩니다.
  - 아래의 예제처럼 `/subscribe` 라는 url 접두사를 구독하고 있는 클라이언트가 있다면, 브로커가 구독중인 클라이언트들을 찾아 메시지를 전달하게 해줍니다. **(Sever to Client)**

```js
stompClient.subscribe('/subscribe/join', function (join) {
    showJoinMessage(JSON.parse(join.body).userName + "님이 입장하셨습니다.");
});
```


- 참고 : 스프링 문서에 따르면, **1:N** 의 경우는 `/topic`, **1:1** 의 경우에는 `/queue` 로 일반적으로 사용하나, 이해를 쉽기위해 `/subscribe` 라는 경로로 지정했습니다.

> The meaning of a destination is intentionally left opaque in the STOMP spec. It can be any string, and it is entirely up to STOMP servers to define the semantics and the syntax of the destinations that they support. It is very common, however, for destinations to be path-like strings where /topic/.. implies publish-subscribe (one-to-many) and /queue/ implies point-to-point (one-to-one) message exchanges.

```java
config.enableSimpleBroker("/topic", "/queue"); 
```

- 위처럼 여러 브로커를 등록할 수도 있습니다.


<br>  


> setApplicationDestinationPrefixes


- `setApplicationDestinationPrefixes(/url)`
- **/url** 로 시작되는 메시지가 서버로 올경우, 메시지를 처리하는 메소드로 메시지를 라우팅하게 합니다.


```java  

// config.class
@Override
public void configureMessageBroker(final MessageBrokerRegistry config) {
    ...
    config.setApplicationDestinationPrefixes("/app");
}

// controller.class
@Controller
public class JoinMessageController {
    @MessageMapping("/join")
    ...
    public JoinResponse joinMessage(final JoinRequest joinRequest) {
        ...
    }
}
```

- `/app/join` 과 같은 형태의 url을 받게 된다면
  - `/app` 을보고 `@MessageMapping` 을 가지고 있는 메소드를 찾게 될것입니다.
    - `/app/**` 는 **@MessageMapping** 어노테이션을 지닌 메소드를 찾습니다.
  - 그 뒤의 `/join` 때문에, `@MessageMapping("/join")` 이 달려있는 메소드(joinMessage)로 최종적으로 메시지 전달이 되겠습니다.

- 결론 : `@MessageMapping("/join")` 은 `app/join` 를 지닌 url이 들어오게 됩니다.


<br>

```js
// client

function sendName() {
    stompClient.send("/app/join", {}, JSON.stringify({'name': $("#name").val()}));
}
```

- 위와같이 클라이언트에서 보내는 메시지를 소화할 수 있게 되어집니다.


---

<br>

전체적인 패키지 구조는 아래와 같이 되겠습니다.  


![image](https://user-images.githubusercontent.com/43930419/129902743-db61ca96-8d46-409f-bd70-3a432ea87eb9.png)

---

## 실행
- 서버를 구동시킨뒤 `http://localhost:8080/` 에 접속합니다.

![image](https://user-images.githubusercontent.com/43930419/129903036-ee5a6f9d-bc32-4d9b-b845-8d58253280d8.png)

- connect를 눌러, 웹소켓 커넥션을 만듭니다.

---

<br>

![image](https://user-images.githubusercontent.com/43930419/129903252-db886f75-9bcb-4a36-a93f-a69b96a9f1f6.png)

- 다른쪽 클라이언트에서도 제대로 표시되는지 확인하기 위해서, 한개의 브라우저를 더 킨다음 
- 똑같이 connect를 눌러, 웹소켓 커넥션을 만듭니다.

---

<br>

![image](https://user-images.githubusercontent.com/43930419/129903300-48ec6b97-a57d-4303-9c38-704290d46970.png)

- 메시지를 보내보면, 다른쪽 클라이언트에서도 메시지가 잘 수신되는것을 확인할 수 있습니다.

---

<br>


![image](https://user-images.githubusercontent.com/43930419/129903338-957fca33-c0b3-4555-b346-6f53244e8623.png)

- 아래쪽 클라이언트에서도 입장하는경우.

---

<br>


### 크롬 개발자 도구
> F12

![image](https://user-images.githubusercontent.com/43930419/129903389-05fce390-1d26-4a03-a637-ae92d9a68d34.png)

- `F12`를 누르면, 어떤식으로 메시지를 주고받는지 확인이 가능합니다.


---

<br>


![image](https://user-images.githubusercontent.com/43930419/129903436-e6143374-a580-4df3-b11a-6e0e0319cfff.png)

- `Network` -> `WS` 로 들어가면, 웹소켓이 커넥션이 한개 만들어져있는것을 볼 수 있고, 해당 커넥션을 통해 어떻게 어떤 메시지를 주고받았는지 확인할 수 도 있습니다. 


> 다음 글은 지금처럼 한개의 방에서 입장, 퇴장 메시지만 전송하는것이 아닌,  
> N개의 방에서 채팅을 주고받도록 하는법을 알아보겠습니다.

---

## 전체 코드

- 전체 코드는 [해당 깃헙](https://github.com/unluckyjung/spring-webSocket-tutorial/tree/step1-join/web-socket)에서 볼 수 있습니다.  

---

## Reference
- https://spring.io/guides/gs/messaging-stomp-websocket
- https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#websocket
- https://docs.spring.io/spring-framework/docs/4.3.x/spring-framework-reference/html/websocket.html