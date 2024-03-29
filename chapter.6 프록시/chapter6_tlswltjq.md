# 06. 프락시
웹 프락시 서버는 중개자, 서버와 클라이언트 사이에서 HTTP메시지를 정리하는 중개인처럼 움직인다.
이 장에서는
- HTTP 프락시와 웹 게이트웨이를 비교하고 HTTP 프락시가 어떻게 배치되는지 그림으로 보여주면서 설명한다.
- 몇 가지 유용한 활용법을 보여준다.
- 프락시가 실제 네트워크에 어떻게 배치되어 있는지, 트래픽이 어떻게 프락시 서버로 가게 되는지 설명한다.
- 브라우저에서 프락시를 사용하려면 어떻게 설정해야하는지 보여준다.
- HTTP 프락시 요청이 서버 요청과 어떻게 다른지, 그리고 프락시가 어떻게 브라우저의 동작을 미묘하게 바꾸는지 보여준다.
- 일련의 프락시 서버들을 통과하는 메시지의 경로를, Via헤더와 TRACE메서드를 통해 기록하는 방법을 설명한다.
- 프락시에 기반한 HTTP 접근 제어를 설명한다.
- 어떻게 프락시가 클라이언트와 서버 사이에서 각각의 다른 기능과 버전 들을 지원하면서 상호작용하는지 알아본다
## 6.1 웹 중재자
<img width="649" alt="image" src="https://github.com/Tobystudy/Http-Study/assets/41179427/01e2e6a0-80f9-4175-8034-95f80b9e4845">

HTTP 프락시서버는 중개인, 웹 클라이언트, 웹 서버.<br> 프락시는 클라이언트로부터 HTTP요청을 받게되고 웹서버처럼 요청과 커넥션을 적절히 다루며 응답을 돌려줘야한다. 동시에 요청을 서버로 보내기도하고 다시 응답을 받는 HTTP 클라이언트와같이 동작을 해야한다. 
### 6.1.1 개인 프락시와 공유 프락시
프락시 서버는 하나의 클라이언트가 독점적으로 사용할 수도 여러 클라이언트가 공유할 수 도 있다.

#### 개인 프락시
흔하지않지만 클라이언트 컴퓨터에서 직접 실행되는 형태로 꾸준히 사용되고있다.<br>ISP와같은 서비스

#### 공용 프락시
대부분의 프락시는 공용 프락시.<br> 중앙 집중형 프락시를 관리하는게 비용효율이 높고 쉬우며 캐시 프락시와 같은경우 공통된 요청에서 이득을 취할 수 있다.

### 6.1.2 프락시 대 게이트웨이
프락시와 게이트웨이의 차이점?
>프락시는 같은 프로토콜을 사용하는 둘 이상의 어플리케이션을 연결<br>게이트웨이는 서로 다른 프로토콜을 사용하는 둘 이상을 연결한다.

## 6.2 왜 프락시를 사용하는가?
보안을 개선하고 성능을 높이고 비용을 절약한다.<br>
HTTP 트래픽을 들여다보고 건드릴 수 있기때문에 여러기능을 하는 웹 서비스를 제공할 수 있다.
- 필터링 프락시
- 문서접근 제어자<br>클라이언트에 따른 다양한 제한
- 보안 방화벽<br> 조직안에 들어오거나 나가는 프로토콜의 흐름을 한 지점에서 통제할 수 있다.
- 웹 캐시
- 대리/리버스 프락시<br>공용 컨텐츠에 대한 느린 웹서버의 성능을 개선하기위해 사용할 수 있다.
- 콘텐츠 라우터<br>조건에 따라 특정 웹서버로 유도하는 라우터로 작동가능
- 트랜스코더<br>클라이언트에 전달하기전 본문포맷을 수정할 수 있다. 이런 데이터표현방식을 자연스럽게 변환하는것을 트랜스 코딩이라고한다.
- 익명 프락시<br> 신원을 식별할 수 있는 특성들을 제거하여 개인정보 보호와 익명성에 기여한다.

<img width="517" alt="image" src="https://github.com/Tobystudy/Http-Study/assets/41179427/a910b518-666b-4d9e-8105-de04c1c41c8b">

클라우드 프론트

## 6.3 프락시는 어디에 있는가?
### 6.3.1 프락시 서버 배치
### 6.3.2 프락시 계층
<img width="649" alt="Screenshot 2024-03-18 at 9 28 19 AM" src="https://github.com/Tobystudy/Http-Study/assets/41179427/d9f90878-ba51-4da9-ba84-16ac9f577026">

>프락시는 원서버에 도착할 때 까지 프락시들로 이루어진 연쇄, 프락시 계층을 통과한다.<br>
>서버에서 가까운쪽(인바운드)를 부모라고 부르고, 먼 쪽(아웃바운드)을 자식이라고 한다.

### 6.3.3 어떻게 프락시가 트래픽을 처리하는가
<img width="645" alt="image" src="https://github.com/Tobystudy/Http-Study/assets/41179427/fa327c15-365c-4eb1-8f31-b736cc7f3084">

클라이언트 트래픽이 프락시로 가도록 만들려면?
- 클라이언트를 수정한다 : 옵션에서 자동 프락시 설정을 끈다.
- 네트워크를 수정한다 : 스위칭/라우팅 장비를 이용해 프락시로 보낸다. 이러한 방식을 인터셉트 프락시라고 한다.
- DNS이름공간을 수정한다 : DNS 이름 테이블을 수동으로 편집하거나 동적 DNS서버를 이용해 조정될 수 있다.
- 웹서버를 수정한다 : HTTP 리다이렉션 명령을 이용해 프락시로 리다이렉트하도록 할 수 있다.

## 6.4 클라이언트 프락시 설정
- 수동 설정 : 프락시를 사용하겠다고 명시적으로 설정한다.
- 브라우저 기본 설정 : 브라우저 벤더나 배포자는 브라우저를 소비자에게 전달 하기 전에 프락시를 미리 설정해 놓을 수 있다.
- 프락시 자동 설정(Proxy auto-configuration, PAC) : 자바스크립트 프락시 자동 설정 파일에 대한 URI 제공을 통해, 언제 써야 하는지, 어떤 프락시를 써야 하는지 실행할 수 있다.
- WPAD 프락시 : 자동설정 파일을 다운받을 수 있는 '설정 서버'를 자동으로 찾아주는 자동발견 프로토콜을 제공한다.
### 6.4.1 클라이언트 프락시 설정: 수동
### 6.4.2 클라이언트 프락시 설정: PAC 파일
### 6.4.3 클라이언트 프락시 설정: WPAD
## 6.5 프락시 요청의 미묘한 특징들
- 프락시 요청의 URI는 서버 요청과 어떻게 다른가.
- 인터셉트 프락시와 리버스 프락시는 어떻게 서버 호스트 정보를 알아내기 어렵게 만드는가.
- URI수정에 대한 규칙
- 프락시는 브라우저의 똑똑한 URI 자동완성이나 호스트 명 확장 기능에 어떻게 영향을 주는가.
### 6.5.1 프락시 URI는 서버 URI와 다르다
클라이언트가 프락시 대신 서버로 요청을 보내면 요청의 URI가 달라진다.

클라이언트가 웹 서버로 요청을 보내면 다음과같이 부분 URI를 가진다.
``` HTTP
GET /index.html HTTP/1.0
User-Agent: SuperBrowser1.3
```
그러나 클라이언트가 프락시로 요청을 보낼 때 요청줄은 다음과 같이 생략하지 않은 완전한 URI를 가진다.
``` HTTP
GET http://www.marys-antiques.com/index.html HTTP/1.0
User-Agent: SuperBrowser1.3
```
왜 다른 요청 형식을 가질까? 원래의 HTTP설계에서 자신의 호스트명과 포트번호를 알고 있으므로 불필요한 정보 발송을 피하기위해 스킴과 호스트가 없는 부분 URI만 보냈지만 프락시가 부상하며 목적지를 알아야하기에 그 서버의 이름을 알 필요가 생겨난것이다.

- 클라이언트가 프락시를 사용하지 않도록 설정되어있다면 부분 URI를 보낸다.
- 클라이언트가 프락시를 사용하도록 설정되어 있다면, 완전한 URI를 보낸다.

### 6.5.2 가상 호스팅에서 일어나는 같은 문제
### 6.5.3 인터셉트 프락시는 부분 URI를 받는다
앞선 케이스의 경우 프락시라는것을 인식하고 완전한URI를 보내었다.<br>
대리/인터셉트 프락시의 경우 눈에 보이지않기에 부분 URI를 받게될것이다.
### 6.5.4 프락시는 프락시 요청과 서버 요청을 모두 다룰 수 있다.
다목적 프락시 서버는 완전 URI와 부분 URI를 모두 지원해야 한다.
완전 URI와 부분 URI를 사용하는 규칙은 다음과 같다.

- 완전한 URI가 주어졌다면, 프락시는 그것을 사용해야 한다.
- 부분 URI가 주어졌고 Host 헤더가 있다면, Host 헤더를 이용해 원 서버의 이름과 포트 번호를 알아내야 한다.
- 부분 URI가 주어졌으나 Host 헤더가 없다면, 다음의 방법으로 원 서버를 알아내야 한다
    1. 대리 프락시라면 프락시에 실제 서버의 주소와 포트번호가 설정되어있을 수 있다.
    2. 이전에 어떤 인터셉트 프락시가 가로챈 트래픽을 받았고 그 인턴셉트 프락시가 원 IP 주소와 포트번호를 사용할 수 있도록 해두었다면, 그 IP 주소와 포트번호를 사용할 수 있다.
    3. 모두 실패했다면, 에러메시지를 반환한다.
### 6.5.5 전송 중 URI 변경
프락시 서버는 요청 URI의 변경에 신경써야하는데 무해해보이는 변경이라도 상호운용성 문제를 일으킬 수 있기에 조심해야하며 관대하도록 애써야한다.<br>
일반적인 인터셉트 프락시가 URI를 전달할 때 빈 경로를 '/'로 교체하는것 이외에는 절대경로를 고쳐쓰는것을 금지한다.
### 6.5.6 URI 클라이언트 자동확장과 호스트 명 분석(Hostname Resolution)
브라우저는 프락시 존재 여부에 따라 요청 URI를 다르게 분석한다.<br>
프락시가 없다면 사용자가 타이핑한 URI를 가지고 그에 대응하는 IP주소를 찾는다 호스트가 발견되지 않으면 다음 몇 가지 시도를 한다.
- 일반적인 웹 사이트 이름의 가운데 부분만 입력했다면, 'www'를 붙이고 '.com'을 붙인다.
- 몇몇 브라우저는 해석할 수 없는 URI를 서드파티 사이트로 넘기기도 하는데, 오타 교정을 시도하고 사요자가 의도했을 URI를 제시한다.
- DNS는 호스트 명의 앞부분만 입력하면 자동으로 도메인을 검색하도록 설정 되어 있다. 'oreilly.com'에서 'host7'을 입력하면 자동으로 'host7.oreilly.com'을 찾아본다.

### 6.5.7 프락시 없는 URI 분석(URI Resolution)
### 6.5.8 명시적인 프락시를 사용할 때의 URI 분석
### 6.5.9 인터셉트 프락시를 이용한 URI 분석
## 6.6 메시지 추적
프락시가 많아지고 흔해지며 메시지는 많은 프락시를 지나치게되어 메시지의 흐름을 추적하고 문제점을 찾는 일도 중요해졌다.
### 6.6.1 Via 헤더
Via 헤더 필드는 메시지가 지나는 각 중간 노드(프락시/ 게이트웨이)의 정보를 나열한다.
#### Via가 개인정보 보호와 보안에 미치는 영향
프락시 서버가 네트워크 방화벽의 일부인 경우 방화벽 뒤에 호스트들의 이름과 포트번호를 전달한다면 네트워크 아키텍처에 대한 정보가 악의적으로 이용될 수 있다.<br>
그렇다면 적당한 가명으로 교체해야한다. 이러한 일로 인해 실제이름을 알기 힘들어져도 Via경유지 항목을 유지하려 노력해야한다.

### 6.6.2 TRACE 메서드
#### Max-Forwards
일반적으로 TRACE메서드는 모든 경로를 여행하는데 이 헤더를 이용해 루프에 빠지지않도록 테스트할 수있다.

## 6.7 프락시 인증
접근 제어 장치로 제공될 수 있다.<br>적절한 접근 권한 자격을 제시하지 않는 다면 콘텐츠에 대한 요청을 차단할 수 있다.
- 제한된 컨텐츠에 대한 요청이 프락시 서버에 도착했을 때, 프락시 서버는 407 상태 코드를 어떻게 그러한 자격을 제출할 수 있는지 설명해주는 Proxy-Authenticate 헤더 필드와 함께 반환 할 수 있다
- 407 응답을 받으면, 로컬 데이터베이스를 확인해서든 사용자에게 물어봐서든 요구되는 자격을 수집한다.
- 자격을 획득하면, 클라이언트는 요구되는 자격을 Proxy-Authorization 헤더 필드에 담아서 요청을 다시 보낸다.
- 자격이 유효하다면, 프락시는 원 요청을 연쇄를 따라 통과시킨다. 유효하지 않다면 407 응답을 보낸다.

## 6.8 프락시 상호 운용성
앞서 말했듯이 프락시, 서버, 클라이언트들은 HTTP명세를 따르지만 여러 벤더에 의해서 만들어졌다. 여러 기능이 있고 제각기의 버그를 가지고 있다. 각 프락시 서버는 서로 다른 프로토콜을 구현할 수도, 이상한 동작을 할 수 도 있는 클라이언트와 서버 사이를 중개해야만 한다.

### 6.8.1 지원하지 않는 헤더와 메서드 다루기
프락시는 이해 할 수 없는 헤더 필드는 반드시 그대로 전달해야 하며, 여러개 일 경우 순서도 반드시 유지해야한다.<br>
지원하지 않는 메서드를 통과시킬 수 없는 프락시는 대부분의 네트워크에서 살아남지 못한다.<br>
HTTP/1.1은 메서드를 확장하는 것을 허용하고 있다.

### 6.8.2 OPTIONS: 어떤 기능을 지원하는지 알아보기
### 6.8.3 Allow 헤더
## 6.9 추가정보
