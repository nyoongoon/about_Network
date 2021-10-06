# about_Network
네트워크를 공부하며 알게된 것을 기록하는 저장소입니다.

# CORS(Cross-Origin Resource Sharing 교차 출처 리소스 공유)
: Origin(출처)란? -> Protocol, host, port를 모두 합친 것. 
: 포트번호는 생략이 가능. (http, https의 포트번호가 정해져 있기 때문)
- SOP(Same-Origin-Policy)
: 같은 출처에서만 리소스를 공유할 수 있다는 규칙. -> CORS정책을 지킨 리소스 요청은 허용.
- 같은 출처?
: 스킴 | 호스트 | 포트 -> 가 같으면 같은 출처로 인정 (http:// | naver.com | (port))
- 출처를 비교하는 로직은 브라우저에 구현되어 있는 스펙임. -> 서버는 정상적으로 응답하나 브라우저가 CORS정책을 판단하여, 서버가 보낸 응답을 그냥 버리게 됨.

- CORS의 동작?
: 기본적으로 웹 클라이언트 어플리케이셔션이 다른 출처의 리소스를 요청할 때는 http 프로토콜을 사용하여 요청을 보내는데, 이 때 브라우저는 요청 헤더에 Origin이라는 필드에 요청을 보내는 출처를 함꼐 담아 보냄.
``` 
Origin: https//naver.com
```
: 이후 서버가 이 요청에 대한 응답으로 응답 헤더의 Access-Control-Allow-Origin이라는 값에 "이 리소스를 접근하는 것이 허용된 출처"를 내려주고, 브라우저는 Origin과 이를 비교한다. 

## 비교하는 세 가지 방식
### 1 Preflight Request
- 프리 플라이트 방식은 예비 요청(Preflight)과 본 요청으로 나누어서 서버에로 전송.<br/>
- 예비 요청은 HTTP 메소드 중 OPTIONS 메소드를 이용<br/>
- js의 fetch() api를 사용하여 브라우저에게 리소스를 받아오라는 명령을 내리면 브라우저는 서버에게 예비 요청을 먼저 보내고, 서버는 이에 대한 응답으로 자신이 어떤것을 허용하고 금지하는지 정보를 응답헤더에 담아 브라우저에게 보내준다.<br/>
- 이후 브라우저는 예비요청과 응답을 비교한 후, 요청을 보내는 것이 안전하다고 판단되면 엔드포인트로 다시 본 요청을 보낸다. 서버가 본 요청에 대한 응답을 하면 브라우저는 최종적으로 이 응답을 js에 넘긴다.<br/>

ex) 
예비 요청 : Origin:https://naver.com <br/>
응답 : Access-Control-Allow-Origin: https://evanmoon.tistory.com <br/>
-> CORS 정책이 위반되어 에러 반환.

### 2 Simple Request
- 예비요청 없이 바로 본 요청을 보낸 뒤, 서버가 이에 대한 응답의헤더에 Access-Control-Allow-Origin과 같은 값을 보내주면 그때 브라우저가 CORS정책을 검사. -> Simple Request를 하기 위한 설계 조건이 까다로움.

### 3 Credentialed Request 
- 인증된 요청을 사용하는 방법.<br/>
- 기본적으로 브라우저가 제공하는 비동기 리소스 요청인 API인 XMLHttpRequest 객체나 fetch API는 별도의 옵션 없이 브라우저의 쿠키 정보나 인증과 관련된 헤더를 함부로 요청에 담지 않음. -> 요청에 인증과 관련된 정보를 담을 수 있게 해주는 옵션이 credentials 옵션.<br/>
- 3가지 옵션 값. same-origin(기본값-같은 출처 간 요청에만 인증 정보를 담을 수 있음), include(모든 요청에 인증 정보를 담을 수 있음), omit(모든 요청에 인증 정보를 담지 않음)<br/>
- include시 두가지 cors룰 추가됨. 1. Access-Control-Allow-Origin에는 \*사용할 수 없으며 명시적인 URL이어야 함. 2. 응답헤더에는 반드시 Access-Control-Allow-Credentials: true가 존재해야 함.<br/>

## CORS를 해결하기 
### Access-Control-Allow-Origin 세팅하기<br/>
- 와일드 카드인 \*을 사용하여 헤더에 세팅하게 되면 모든 요청을 받을 수 있지만 보안 이슈가 생길 수 있으니 특정 출처를 명시하는 게 좋다. <br/>
- Nginx나 Apache같은 서버 엔진의 설정에서 추가할 수도 있지만, 소스 코드 내에서 응답 미들웨어 등을 사용하여 세팅하면 좋다. (백엔드 프레임 워크의 미들웨어 라이브러리)<br/>

### Webpack Dev Server로 리버스 프록싱 하기<br/>
- 웹팩과 webpack-dev-server 라이브러리가 제공하는 프록시 기능을 사용하면 CORS정책 우회 가능

출처 : https://evan-moon.github.io/2020/05/21/about-cors/
