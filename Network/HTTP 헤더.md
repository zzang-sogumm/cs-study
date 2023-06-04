# 💡 HTTP 헤더
HTTP 헤더는 HTTP 전송에 필요한 모든 부가정보를 담고 있다.      
ex) 메시지 바디 내용/크기, 압축, 인증, 요청 클라이언트, 서버 정보 등등        
표준 헤더는 매우 다양하므로, 필요 시마다 [표준 헤더](https://en.wikipedia.org/wiki/List_of_HTTP_header_fields)에서 찾아서 사용하는 것이 좋다.


## 개요
2014년 RFC7230 ~ 7235 등장 이후 기준으로 헤더를 분류하면 다음과 같다.

- General 헤더: 메시지 전체에 적용되는 정보
- Representation 헤더: 표현 데이터를 해석할 수 있는 정보 정보
- Request 헤더: 요청 정보
- Response 헤더: 응답 정보

### 📍 General 헤더
#### Connection
- close: 메시지 교환 후 TCP 연결 종료
- keep-alive: 메시지 교환 후 TCP 연결 유지


### 📍 Representation 헤더
#### Content-Type
- 표현 데이터의 형식 설명
- ex) Content-Type: application/json
- ex) Content-Type: text/html; charset=utf-8

#### Content-Encoding
- 표현 데이터를 압축하기 위해 사용
- 데이터를 전달하는 곳에서 압축 후 인코딩 헤더 추가
- 데이터를 읽는 쪽에서 인코딩 헤더의 정보로 압축 해제
- ex) Content-Encoding: gzip
- ex) Content-Encoding: identify

#### Content-Language
- 표현 데이터의 자연 언어를 표현
- ex) Content-Language: ko
- ex) Content-Language: en

#### Content-Length
- 표현 데이터의 길이
- Transfer-Encoding(전송 코딩)을 사용하면 Content-Length를 사용하면 안 됨
- ex) Content-Length: 3423




### 📍 Request 헤더
#### 협상
- 클라이언트가 선호하는 표현 요청
- 협상 헤더는 요청 시에만 사용함
- 예를 들어, 다중 언어를 지원하는 서버에 Accept-Language에 ko를 담아 보내면 그것에 맞춰서 response를 보냄
- Quality Value(q)를 사용한 우선 순위가 적용됨

- Accept
  - 클라이언트가 선호하는 미디어 타입 전달 
- Accept-Charset
  - 클라이언트가 선호하는 문자 인코딩
- Accept-Encoding
  - 클라이언트가 선호하는 압축 인코딩
- Accept-Language
  - 클라이언트가 선호하는 자연 언어

#### Referer
- 현재 요청된 페이지의 이전 웹페이지 주소를 나타냄
- 유입 경로를 분석할 수 있음
- referrer의 오타

#### User-Agent
- 클라이언트의 애플리케이션 정보(웹 브라우저 정보 등)를 나타냄
- 어떤 종류의 브라우저에서 장애가 발생하는지 파악 가능
- ex) User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/ 537.36 (KHTML, like Gecko) Chrome/86.0.4240.183 Safari/537.36

#### Host
- 요청한 호스트 도메인 정보(필수)
- 하나의 서버가 여러 도메인을 처리해야 할 때, 혹은 하나의 IP 주소에 여러 도메인이 적용되었을 때 사용

#### 인증
- Authorization
  - 인증 토큰을 서버로 보낼 때 사용하는 헤더
  - ex) Authorization: Basic xxxxxxxxxxxxxxxx
- WWW-Authenticate
  - 리소스 접근 시 필요한 인증 방법 정의
  - 401 Unauthorized 응답과 함께 사용
  - ex) WWW-Authenticate: Newauth realm="apps", type=1,  
    title="Login to \"apps\"", Basic realm="simple"

#### Cookie
- 클라이언트가 서버에서 받은 쿠키를 저장하고, HTTP 요청 시 서버로 전달





### 📍 Response 헤더
#### Date
- HTTP 메시지를 생성한 일시
- ex) Date : Sun, 02 June 2023 08:26:08 GMT

#### Server
- 요청을 처리하는 origin 서버의 소프트웨어 정보
- ex) Server: nginx

#### Location
- 페이지 리다이렉션
- Location 헤더가 있으면 Location 위치로 자동으로 이동함
- HTTP 상태 코드 3xx에서 사용됨
- 201 (Created)의 Location 값은 요청에 의해 생성된 리소스 URI
- 3xx (Redirection)의 Location 값은 요청을 자동으로 리디렉션하기 위한 대상 리소스를 가리킴

#### Allow
- 서버 측에서 지원 가능한 HTTP 메서드의 리스트
- 405 (Method Not Allowed)에서 응답에 포함해야함
- ex) Allow: GET,POST

#### Set-Cookie
- 서버에서 클라이언트로 쿠키 전달
- 쿠키는 사용자 로그인 세션을 관리하거나, 광고 정보를 트래킹 하는 데에 주로 사용됨
- 쿠키 정보는 항상 서버에 전송됨
  - 네트워크 트래픽 추가 유발
  - 최소한의 정보(세션 id, 인증 토큰)만 사용
- 서버에 전송하지 않고, 웹 브라우저 내부에 데이터를 저장하고 싶으면 웹 스토리지 (localStorage, sessionStorage) 에 저장 
- ex) set-cookie: sessionId=abcde1234; expires=Sat, 26-Dec-2020 00:00:00 GMT; path=/; domain=.google.com; Secure


- expires
  - 만료일이 되면 쿠키 삭제 
  - ex) expires=Sat, 26-Dec-2023 22:39:21 GMT
- max-age
  - 최대 유지 기간
  - 0이나 음수를 지정하면 쿠키 삭제
  - max-age=3600 -> 3600초간 유지
- 세션 쿠키
  - 만료 날짜를 생략하면 브라우저 종료 시까지만 유지
- 영속 쿠키 
  - 만료 날짜를 입력하면 해당 날짜까지 유지

- domain 
  - ex) domain=example.org
  - 명시하는 경우
    - 명시한 문서 기준 도메인 + 서브 도메인 포함
    - ex) example.org는 물론, dev.example.org도 쿠키 접근
  - 생략하는 경우
    - 현재  문서 기준 도메인만 적용
    - ex) example.org 에서만 쿠키 접근
 
- path
  - 이 경로를 포함한 하위 경로 페이지만 쿠키 접근 
  - 일반적으로 path=/ 루트로 지정
  - ex) path =/home 으로 지정하면,
    - `/home`, `/home/level1` -> 가능
    - `/hello` -> 불가능

- Secure
  - secure를 적용하면 https인 경우에만 전송
  - 일반적으로 쿠키는 http, https를 구분하지 않지만 해당 옵션을 사용하면 https인 경우에만 전송
- HttpOnly
  - XSS 공격 방지
  - 자바스크립트에서 접근 불가(document.cookie)
  - HTTP 전송에만 사용
- SameSite
  - XSRF 공격 방지
  - 요청 도메인과 쿠키에 설정된 도메인이 같은 경우만 쿠키 전송



-------------------------------------------------


### References
- [김영한 - 모든 개발자를 위한 HTTP 웹 기본 지식](https://www.inflearn.com/course/http-%EC%9B%B9-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC)
- https://developer.mozilla.org/ko/docs/Web/HTTP/Messages
- https://code-lab1.tistory.com/257