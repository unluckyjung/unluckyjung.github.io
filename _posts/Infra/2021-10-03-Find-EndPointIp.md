---
title: Reverse Proxy를 거치기전의 IP를 WAS에서 알아내기
date: 2021-10-03-02:40
categories: 
- Infra

tags:
- Nginx
- Web Server
- WAS
- Proxy
- Reverse Proxy

---

## Reverse Proxy를 거치기전의 IP를 찾기
> Reverse Proxy를 거치기전의 EndPoint Clinet의 IP를 WAS에서 알아내 봅니다.

---

## Goal
- Reverse Proxy를 거치기전의  Clinet의 IP를 WAS에서 알수 있도록 구성해봅니다.  

---

## 개요
> IP 차단 기능을 구현했는데, 배포환경에선 정상적으로 작동하지 않았습니다.

![reverse-proxy](https://user-images.githubusercontent.com/43930419/135726921-79ffea1d-aee9-4cda-b7e9-5b7ff6a0c65d.jpg)


원인은, WAS측에서 요청한 클라이언트의 IP를 보고 필터링을 하도록 구현했는데, WAS는 실제 Client가 아닌, 본인입장에서 Client인 중간에 있는 리버스 프록시(NGINX)의 IP를 알게되기 때문입니다.

---

## 해결
> 리버스 프록시에서 X-Forwarded-For Header 추가

- X-Forwarded-For Header 는 엄밀하게는 표준이 아니지만, 이런 상황에 대해서 사실상 표준처럼 쓰이는 헤더입니다.
- 리버스 프록시(NGINX)쪽에서 `X-Forwarded-For` Headerd의 값으로 **클라이언트 IP**를 넣어줍니다.
- WAS측에선 X-Forwarded-For 헤더에 담겨있는 IP를 확인하도록 수정합니다.


### NGINX 설정 추가

- NGINX에서는 `$proxy_add_x_forwarded_for` 라는 변수를 제공합니다.
- 클라이언트의 IP 주소를 담고있는 `$remote_addr` 를 `X-Forwarded-For` Header의 값으로 추가해 주게 됩니다.

```java
// nginx.conf
...

location / {
  proxy_pass http://app/;
  proxy_http_version 1.1;
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection "upgrade";
  proxy_set_header Host $host;

  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  # 이부분을 추가해주었다.
}
...

```

### WAS 로직 수정

```java
private String getClientIpFrom(final HttpServletRequest request) {
    String clientIp = request.getHeader(X_FORWARDED_FOR_HEADER);

    if (Objects.isNull(clientIp)) {
        clientIp = request.getRemoteAddr();
    }

    return clientIp;
}
```

---

## Conclusion
- `X_FORWARDED_FOR_HEADER` 를 이용해 EndPoint Clinet의 IP를 얻을 수 있다.
- 최초 요청을 보내는 클라이언트에서는 추가적인 작업이 전혀 필요없다. NGINX에서의 설정, WAS의 구현만 살짝 바꾸어주면 해결되었다.

---

## Reference
- https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/X-Forwarded-For
- https://www.nginx.com/resources/wiki/start/topics/examples/forwarded/