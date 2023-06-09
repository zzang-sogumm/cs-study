# 🐲 HTTP 검증 헤더와 프록시 캐시
웹에서 만약 캐시를 사용하지 않는다면, 데이터가 변경되지 않아도 계속 네트워크를 통해서 데이터를 다운로드 받아야 할 것이다.       
그런데 인터넷 네트워크는 매우 느리고 비싸기 때문에 문제가 발생한다.      
브라우저 로딩 속도도 느리고 결국 느린 사용자 경험을 일으킬 수 밖에 없다.

따라서 캐시를 적용하여 캐시 가능 시간동안 네트워크를 사용하지 않아도 되도록 개선한다.      


## 검증 헤더 (Validator)
캐시 만료 후에도 서버에서 데이터를 변경하지 않았으면 재사용할 수 있도록,       
클라이언트의 데이터와 서버의 데이터가 같다는 사실을 확인하는 방법으로 검증 헤더를 사용한다.

### 1. Last-Modified
- 데이터가 마지막으로 수정된 시간 정보를 헤더에 작성      
ex) Last-Modified: Thu, 04 Jun 2020 07:19:24 GMT
- 응답 결과를 캐시에 저장할 때 데이터 최종 수정일도 저장됨

#### 📍 If-Modified-Since, If-Unmodified-Since
- 캐시가 만료되어 다시 요청할 때, 캐시에 Last-Modified 가 있다면, 요청 헤더의 `if-modified-since`에 해당 날짜를 담아서 서버에 보냄
- 서버의 최종 수정일과 비교해서 
  - 데이터가 수정되지 않은 경우 `304 Not Modified`로 알려줌
  - 수정된 경우 `200 OK`로 알려줌
- 네트워크 다운로드가 발생하지만, 용량이 적은 헤더 정보만 다운로드 하므로 **효율적**이다.

#### 📍 단점
- 1초 미만 단위로 캐시 조정이 불가능함
- 날짜 기반의 로직을 사용함
- 데이터를 수정하고 다시 원래의 데이터로 수정하여, 날짜는 다르지만 수정한 데이터 결과가 같은 경우에 낭비됨
- 서버에서 별도의 캐시 로직을 관리하기 어려움
  - 크게 영향이 없는 수정은 캐시를 유지하고 싶은 경우 등

### 2. ETag (Entity Tag)
- 캐시용 데이터에 임의의 고유한 버전 이름을 달아두는 방법
- 캐시 제어 로직을 서버에서 관리       
ex) ETag: "v1.0", ETag: "asid93jkrh2l"
- 데이터가 변경되면 버전을 바꾸어서 변경함(Hash를 다시 생성)
- ETag만 전송하여 같으면 유지, 다르면 다시 다운로드 받음

#### 📍 If-Match, If-None-Match
- 캐시가 만료되어 다시 요청할 때, 캐시에 ETag 가 있다면, 요청 헤더의 `If-None-Match`에 담아서 서버에 보냄
- ETag가 동일한 경우(수정되지 않은 경우), `If-None-Match`실패
  - `304 Not Modified`로 응답


## 프록시 캐시 (Proxy Cache) 헤더
프록시는 원 서버를 대리하여 통신하고, `캐시`, `로드밸런서` 등의 중계 역할을 하는 서버를 말한다.   
일반적으로 클라이언트에서 서버로 리소스를 요청할 때, 프록시 서버를 거쳐 요청하게 된다.   

원 서버와 클라이언트 간의 거리가 매우 멀어도 빠른 요청과 응답이 가능한 이유가 바로 `프록시 캐시` 덕분이다.

### 📍 Cache-control (캐시 지시어)
ex) Cache-Control: public, Cache-Control: must-revalidate, ... 
- public
  - 응답이 public 캐시에 저장되어도 됨
- private
  - 응답이 해당 사용자만을 위한 것임
  - private 캐시에 저장해야 함
- s-maxage
  - 프록시 캐시에만 적용되는 캐시 유효 시간(초)
- no-store
  - 데이터에 민감한 정보가 있으므로 저장하면 안됨
- no-cache 
  - 데이터는 캐시해도 되지만, 항상 origin 서버에 검증하고 사용
- must-revalidate
  - 캐시 만료 후 최초 조회 시 원 서버에 검증해야함
  - 원 서버 접근 실패 시 반드시 `504 Gateway Timeout` 오류가 발생해야함

#### 💥 no-cache VS must-revalidate

<img src="images/no-cache.png" width="800" height="350">

- `no-cache`
  - 프록시 캐시 서버에 요청 시 no-cache인 경우 원 서버에 요청하여 검증 후 `304` 응답
  - 원 서버에 접근 불가 시 오류가 아닌 오래된 데이터라도 보여주는 방식으로 `200 OK` 응답

<img src="images/must-revalidate.png" width="800" height="350">

- `must-revalidate`
  - 원 서버 접근 불가 시 `504 Gateway Timeout` 오류 보냄
  - 중요한 정보인 경우, 원 서버가 못 받았다고 예전 데이터를 보내면 문제가 발생하므로, must-revalidate 사용



### 📍 Pragma: no-cache (캐시 제어)
- HTTP 1.0 하위 호환

### 📍 Expires (캐시 만료일 지정)
- 캐시 만료일을 정확한 날짜로 지정
- HTTP 1.0 부터 사용
- 지금은 더 유연한 `Cache-Control: max-age` 권장

### 📍 Age:60 
- 오리진 서버에서 응답 후 프록시 캐시 내에 머문 시간(초)


-------------------------------------------------


### References
- [김영한 - 모든 개발자를 위한 HTTP 웹 기본 지식](https://www.inflearn.com/course/http-%EC%9B%B9-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC)
- https://tigercoin.tistory.com/191