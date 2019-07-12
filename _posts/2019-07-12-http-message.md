---
published: true
layout: single
title: "[HTTP] http 메시지 흐름"
category: TIL
comments: true
---

웹은 **클라이언트** 와 **서버** 의 상호작용이다. 다른 누군가와 통신하려면 통신 규약이 필요하다. HTTP, Hyper Text Transfer Protocol이 웹의 통신 규약(방법)이다.

클라이언트와 서버는 어떻게 상호작용 하고 있을까?

## 메시지의 흐름

### 시나리오

andole은 크롬 브라우저를 켜고 주소창에`github.com` 을 입력한다. 짧은 시간이 지나고 깃헙 메인 화면이 나타났다.

### DNS

Domain Name System 이다. 도메인 네임을 IP주소로 변환해준다.  
터미널을 열고 다음 명령을 실행해 보자.

```bash
// for macOS Or Linux
$ ifconfig | grep inet

// for Windows
$ ipconfig
```

터미널에 나오는 xxx.xxx.xxx.xxx 숫자가 바로 자신 컴퓨터의 **IP** 이다. 

그렇다! 네트워크상 모든 노드(컴퓨터)는 IP 주소를 갖고 있다. 이 말은 `github.com` 역시 4개의 3자리 숫자로 구분된 **IP**로 식별된다는 말이다. 하지만 주소창에 숫자가 아닌 `github.com` 을 입력해도 잘 되는데?

```bash
$ ping github.com
// 15.164.81.167
```

`github.com` 의 IP 주소는 `15.164.81.167` 이다. 주소창에 15.164.81.167 을 입력해 보시라. 무엇이 나오나? `github.com`이 나온다.

상상해보라. 어느 사이트에 들어갈 때마다 4개의 숫자를 입력해야 한다. 우리에게 친숙한 **언어** 로 어떤 **사이트(서버)** 를 가리킬 수 없을까?

특정 IP에 대한 **이름**이 도메인이다. `github.com`, `naver.com`, `google.com` 등이 도메인이다. 도메인은 선점효과가 있기 때문에, **비용을 내고** 도메인 네임 서비스에 등록한다. 도메인에서 가리킬 IP를 등록하면, DNS가 IP로 변환해 준다!

시나리오대로 주소창에 `github.com` 을 입력하면, 브라우저는 먼저 `github.com` 에 대한 `IP` 를 알아내려 한다. `DNS` 에 `github.com` 에 대한 IP주소를 물어보고, 응답받은 IP주소로 요청을 보낸다.

매번 DNS에 물어보면 여러모로 효율적이지 못하니, 한번 방문한 사이트는 IP정보를 보관(캐시)한다.

### 라우터

`github.com` 서버는 아주 멀리 떨어져 있다. _(정확히 어디에 있는지는 모르겠지만, 바다 건너 있는건 확실하다)_ 어떻게 내 컴퓨터에서 요청한 내용을 `github.com` 은 건네줄 수 있을까? 내가 보낸 요청을 `github.com` 은 어떻게 받아보는 것일까?

웹은 인터넷 위에서 동작한다. 인터넷은 어마무시하게 많은 노드가 있는 네트워크다. 내 요청은 네트워크 바다 한 귀퉁이에서 반대편 한 귀퉁이로 이동해야 한다. **라우터**는 네트워크의 한 영역의 패킷을 다른 영역으로 보내는 역할을 한다. 라우터는 OSI 7 계층에서 `3 - Network layer` 에 속한다. 

즉, 내가 보낸 요청은 **바로** `github.com` 에 도착하는게 아니라, **여러번의 라우팅을 거쳐** 도달한다.  
터미널에서 다음 명령을 실행해 보자.

```bash
$ traceroute github.com
// 1. xxx.xxx.xxx.xxx ...
// 2. xxx.xxx.xxx.xxx ...
...
```

`traceroute` 는 라우팅 과정을 추적하는 명령어다. `trace + route` `github.com` 으로 보낸 메시지가 여러 라우터를 거쳐가는 것을 확인할 수 있을 것이다.



### 메시지 손실

`github.com` 과 내 컴퓨터 사이 통신 과정에는 굉장히 많은 단계가 있다. 이 중간 과정 중에서 **메시지 손실 (또는 잡음)** 이 발생할 수 있다.  
메시지 손실 없이 정보를 교환하려면 어떻게 해야 할까? 상대가 내 말을 **잘 듣고 있다는 확신** 이 있을 때 메시지를 보내는 건 어떨까?  
이 아이디어를 구체화한게 **TCP**다. 

TCP Transmission Control Porotocol 전송 제어 프로토콜이다. OSI 7 계층의 4번째로, `Transport Layer`, 전송 계층으로 분류된다.  
TCP는 신뢰성 있는 통신을 보장한다. TCP 커넥션은 상대가 내 메시지를 잘 받고 있는지 확인하고 나서 정보를 전송한다. 만약 메시지가 소실되거나 받을 수 없는 상황이면 목적 메시지를 보내지 않는다. TCP에 관해서는 나중에 따로 포스팅하겠다.



### 시나리오 다시 보기

1. andole이 브라우저에서 `github.com` 을 입력하고 엔터를 친다.

2. DNS로부터 해당 IP주소를 얻어온다.
3. IP주소와 포트를 이용, 대상 서버로 TCP 연결을 시도한다. 
4. `andole`이 발송한 메시지는 로컬 네트워크에서 인터넷으로, 다시 `github.com` 로 라우팅된다.
5. `github.com` 은 TCP 커넥션을 인지하고 제대로 받았다는 신호를 andole에게 보낸다.
6. 몇 번의 패킷 교환 후, `andole`과 `github.com` 은 TCP 연결을 수립(ESTABLISHED)한다.
7. `andole` 은 HTTP를 이용하여 리소스를 요청한다.
8. `github.com` 은 HTTP를 이용하여 리소스를 전달한다.
9. 브라우저는 리소스를 받아 화면에 표시한다.

다음 주제는 HTTP 메시지 내부다. 헤더, 바디, 메서드 등...
























