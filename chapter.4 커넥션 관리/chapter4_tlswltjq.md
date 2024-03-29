# 커넥션 관리
이 장에서는
- HTTP가 어떻게 TCP를 사용하는가
- TCP 커넥션의 지연, 병목, 막힘
- 병렬 커넥션, keep-alive 커넥션, 커넥션 파이프라인을 활용한 HTTP의 최적화
- 커넥션 관리를 위해 따라야할 규칙들
## 4.1 TCP커넥션
HTTP는 TCP/IP를 통해 이루어진다.<br>
커넥션이 맺어지고나면 손실, 손상없는 올바른 순서의 전송을 보장한다.<br>
#### 브라우저에서 HTTP 커넥션의 과정
1. 브라우저가 호스트명을 추출한다.
2. 브라우저가 호스트명에대한 IP를 찾는다.
3. 브라우저가 포트번호를 얻는다.
4. 브라우저가 2와 3에서 획득한 IP와 포트번호로 TCP커넥션을 생성한다.
5. 브라우저가 서버로 요청 메시지를 보낸다.
6. 브라우저가 서버로부터 온 HTTP 응답 메시지를 읽는다.
7. 브라우저가 커넥션을 끊는다.

HTTP는 일부 규칙을 제외하면 TCP커넥션에 불과하다.

### 4.1.1 신뢰할 수 있는 데이터 전송 통로인 TCP

### 4.1.2 TCP 스트림은 세그먼트로 나뉘어 IP 패킷을 통해 전송된다.
TCP는 클라이언트로부터 서버로 바이트들을 충돌없이 순서대로 전달한다.<br>
[http, https 스택, ip패킷의 구조 이미지 첨부]

### 4.1.3 TCP 커넥션 유지하기
TCP 커넥션은 네 가지 값으로 식별된다.<br>
<발신지 ip주소, 발신지 포트, 수신지 ip주소, 수신지 포트>
네 가지 값이 모두 같은 커넥션은 있을 수 없다.

### 4.1.4 TCP 소켓 프로그래밍
| 함수 | 설명 |
|---|---|
| socket()| 새로운 소켓을 생성합니다.|
| bind()| 소켓에 로컬 포트와 인터페이스를 할당합니다.|
| connect()| 로컬의 소켓과 호스트의 포트사이에 커넥션을 생성합니다. |
| listen()| 커넥션을 받아들이기 위해 로컬 소켓에 허용함을 표시|
| accept()| 누군가 로컬 포트에 커넥션을 맺기를 기다림|
| read()| 소켓에서부터 버퍼에 n바이트 읽기 시도.|
| write()| 소켓으로부터 버퍼에 n바이트 쓰기 시도.|
| close()| 소켓을 닫습니다.|
| shutdown()| 소켓의 송수신을 중지하거나 연결을 닫습니다. |
| getsockopt()| 소켓 옵션 값을 가져옵니다. |
| setsockopt()| 소켓 옵션을 설정합니다.|

소켓API를 이용해 TCP엔드포인트를 구성, 연결 데이터스트림을 읽고 쓸 수 있다.

## 4.2 TCP 성능에 대한 고려
TCP바로 위에 있기때문에 당연하게 TCP성능의 영향을 받는다.

### 4.2.1 HTTP 트랜잭션 지연
[이미지 첨부]<br>
처리보다 TCP 커넥션을 설정하는 부분이 대부분을 차지한다.때문에 대부분의 HTTP지연은 TCP네트워크의 지연이 원인이다.<br>
#### HTTP 트랜잭션 지연의 원인
1. 방문한적없는 호스트라면 DNS를 이용해 ip주소로 변환하는데 시간이 필요하다.
2. 새롭게 생겨나는 HTTP트랜잭션들
3. 요청 메시지가 인터넷을 통해 전달되고 처리되는데 소요되는 시간들
4. 다시 돌아오는 응답 또한 시간이 소모된다.

### 4.2.2 성능관련 중요 요소
- TCP의 헨드셰이크 설정
- 인터넷 혼잡제어를 위한 slow-start
- 데이터를 한번에 전송하기위한 Nagle 알고리즘
- TCP의 편승(piggyback) 확인응답(acknowledgment)를 위한 확인응답 지연 알고리즘
- TIME_WAIT 지연과 포트고갈

### 4.2.3 TCP 커넥션 핸드셰이크 지연
데이터 전송을 위한 커넥션은 필수적으로 맺어지기때문에 HTTP트랜잭션이 작을수록 상대적으로 비중이 커지게된다.<br>
때문에 존재하는 커넥션을 재활용한다면 효율적일것이다.

### 4.2.4 확인응답 지연
TCP는 신뢰성있는 전송을 보장하지만 TCP가 이용하는 인터넷이 보장하는것이 아니다.<br>
때문에 자체적인 확인 체계를 가진다.<br>
각 TCP 세그먼트는 순번과 데이터 무결성 체크섬을 가지고있기 때문에 온전히 수신하면 확인응답을 반환하고 일정시간안에 그러지 못한다면 다시 전송한다.

####  편승 piggyback
메시지에 대한 응답이나 확인을 받기 위해 별도의 패킷을 보내지 않고, 이미 수신한 메시지에 응답을 실어서 보내는 것<br>
piggyback을 이용해 네트워크를 효율적으로 사용한다.<br>

이를 좀 더 적극적으로 활용하기위해 송출할 확인 응답을 버퍼에 잠깐 저장해두고 확인 응답을 편승시킬 데이터 패킷을 찾아 실어 보낸다.<br>
일정시간안에 데이터 패킷을 찾지못하면 별도 패킷을 만들어 전송된다.<br>

요청과 응답으로 이루어진 HTTP의 동작방식에 따라 확인 응답이 송출 데이터 패킷에 편승할 기회가 적고 편승할 패킷을 찾기으려 한다면 이때문에 지연이 발생하기도 한다.

### 4.2.5 TCP 느린 시작(slow start)
TCP의 전송속도는 TCP의 생성시점에 따라 달라질 수 있다.<br>
생성시 최대 속도를 제한하고 자체적인 튜닝 과정을 거쳐 속도제한을 높혀나가는것을 TCP slow start라고 한다.<br>
slow start는 처음 패킷의 수를 제한해 성곡적으로 전달 되는 시점에 패킷의 양을 늘려간다.<br>
? : 속도를 제한하는것을 sliding window, 패킷의 크기를 제한하는것을 window scaling이라고 생각했는데 그렇지 않은것 같다.

### 4.2.6 네이글(Nagle)알고리즘과 TCP_NODELAY
TCP스택으로 전송하기위해 40바이트 상당의 플래그와 헤더를 포함한다, 따라서 작은 크기의 데이터가 든 패킷을 많이 전송한다면  성능은 크게 떨어질 것이다.<br>
네이글 알고리즘은 패킷을 전송하기전 많은 양의 TCP데이터를 하나로 합쳐 전송한다.
[네이글 알고리즘 그림](https://devjh.tistory.com/106)

### 4.2.7 TIME_WAIT의 누적과 포트고갈
TCP 커넥션의 끝에서 커넥션이 끊기면 커넥션의 ip와 포트번호를 기록해둔다.<br>
이 정보를 이용해 일정 시간동안 동일한 주소와 포트번호를 사용하는 커넥션의 생성을 방지한다.<br>
이것으로 이전 커넥션과 관련된 패킷이 새로운 커넥션에 삽입되는 문제를 방지한다.<br>
방지하는 시간은 2MSL(세그먼트 생명주기의 두배 2분정도)로 짧게 수정할 수 있지만 지양해야한다.

클라이언트가 서버에 접속할 때마다 커넥션을 새로운 포트에 할당한다. 굉장히 빠르다면 서버에 존재하는 소켓이 생성되고 닫힐 것이다. TIME_WAIT인 포트로 가득차고 포트 고갈이 일어날 수 있다.

## 4.3 HTTP 커넥션 관리
### 4.3.1 흔히 잘못 이해하는 Connection 헤더
HTTP에서는 클라이언트와 서버 사이에 많은 중개 서버가 놓이는것을 허락하는데 이 때 호스트와 호스트 사이 어느 한 곳에만 적용해야할 옵션을 지정해야할 때가 있다.<br>이 때 사용하는 필드가 HTTP Connection헤더필드이다.

#### Connection 헤더에 전달 될 수 있는 토큰
 - HTTP 헤더 필드명은, 이 커넥션에만 해당되는 헤더들을 나열한다.
 - 임시적인 토큰값은, 커넥션에 대한 비표준 옵션을 뜻한다.
 - close 값은, 커넥션이 작업이 안료되면 종료되어야 함을 뜻한다.

#### hop-by-hop
메시지를 다른 커넥션에 전달하지 않기 위해 Connection헤더에 기술하는방식.    

### 4.3.2 순차적인 트랜잭션 처리에 의한 지연
순차적인 처리는 물리적, 심리적 지연을 만들어낸다.<br>
HTTP 커넥션의 성능을 향상시키기 위한 기술 네 가지
1. 병렬(parallel) 커넥션 : 여러개의 TCP 커넥션을 통해 동시 HTTP요청
2. 지속(persistent) 커넥션 : 커넥션을 맺고 끊는 과정에서 생기는 지연을 제거하기 위한 TCP커넥션의 재활용
3. 파이프라인(pipelined) 커넥션 : 공유 TCP커넥션을 통한 병렬 HTTP요청
4. 다중(multiplexed) 커넥션 : 요청과 응답에 대한 중재(실험적)

## 4.4 병렬 커넥션
### 4.4.1 병렬 커넥션은 페이지를 더 빠르게 내려받는다
### 4.4.2 병렬 커넥션이 항상 더 빠르지는 않다
병렬 커넥션이 일반적으로 빠르지만 항상 그렇진 않음, 클라이언트의 네트워크 대역폭이 좁다면 여러개의 커넥션을 생성하면서 발생하는 부하와 느린 데이터 전송으로 성능상 장점이 거의 없어진다.<br>
또한 많은 커넥션은 많은 메모리를 사용, 서버 자체적 성능문제를 발생시킨다.<br>
따라서 브라우저는 실제로 병렬 커넥션을 사용하지만 대부분 적은 수의 병렬 커넥션만을 허용한다.

### 4.4.3 병렬 커넥션은 더 빠르게 '느껴질 수' 있다
이는 여러개의 객체가 화면에 표시되면서 얻는 심적 안정감

## 4.5 지속 커넥션
웹 클라이언트는 보통 같은 사이트에 여러개의 커넥션을 맺는다. ex)여러 그림들이 있고 대부분 같은 서버에 요청할 것이다. 이를 site locality라고한다.<br>
그러므로 요청을 처리 후 앞으로의 요청을 위해 웹 클라이언트는 해당 TCP커넥션을 유지해 느린 시작을 피한다.

### 4.5.1 지속 커넥션 vs 병렬 커넥션
지속 커넥션은 병렬 커넥션에 비해 커넥션을 맺고 끊는 시간을 줄이고 튜닝된 커넥션을 유지하여 너켁션의 수를 줄인다.<br>
관리를 잘못하면 수많은 커넥션이 쌓이게 될 수 있다.<br>
두 특성을 함께 활용해 사용하면 적은 수의 병렬 커넥션을 유지해 효율적인 데이터 전송과 커넥션 관리를 할 수 있다.

### 4.5.2 HTTP/1.0+의 Keep-Alive 커넥션
### 4.5.3 Keep-Alive 동작
### 4.5.4 Keep-Alive 옵션
### 4.5.5 Keep-Alive 커넥션 제한과 규칙
#### Keep-Alive 커넥션 제한과 규칙
- HTTP/1.0에서 기본으로 사용되지 않아 요청 헤더를 따로 보내야함
- 이를 계속 유지하려면 모든 메시지에 Connection: Keep-Alive 헤더가 포함되어 있어야 함(없으면 이를 보고 커넥션을 끊을 것임을 추론)
- 끊어지기 전, 엔터티의 본문의 길이를 알 수 있어야 커넥션 유지 가능
- 프락시와 게이트웨이는 Connection 헤더의 규칙을 철저히 지켜야함
- 이 헤더를 인식 못하는 프락시 서버와 맺어지면 안된다.
- HTTP/1.0을 따르는 기기로부터받는 모든 Connection 헤더 필드는 무시해야한다. (오래된 프락시 서버로부터 실수로 전달 될 수 있어서)
- 클라이언트는 응답 전체를 모두 받기 전에 커넥션이 끊어졌을 경우 별다른 문제가 없으면 요청을 다시 보낼 수 있게 준비되어야한다.

### 4.5.6 Keep-Alive와 멍청한(dumb)프락시
keep-alive는 프락시가 모르기때문에 Connection에 명시된 헤더를 전달하면 안된다.

### 4.5.7 Proxy-Connection 살펴보기
홉(각 호스트)별헤더들은 한 개의 특정 커넥션에서 쓰이고 전달되면 안되는데, 프락시는 이를 모르기에 오해를 만들어 내는것이다. 이를 방지하기에 비표준 헤더인 Proxy-Connection확장헤더를 프락시에 전달한다. 지속 커넥션 핸드셰이킹을 이해할 수 있다면 이를 Connection으로 헤더로 변경함으로써 원하던 효과를 얻게된다.

### 4.5.8 HTTP/1.1의 지속 커넥션
HTTP/1.1에서는 keep-alive 커넥션을 지원하지 않지만 개선된 지속커넥션을 지원한다. 목적은 같지만 더 잘 작동한다.<br>
HTTP/1.1에서 이는 기본으로 활성화 되어있고 HTTP/1.1어플리케이션이 다음 커넥션을 끊으려면 Connection: close를 명시 해야한다. 해당 헤더가 없더라도 유지하겟다는 뜻은 아님. 추정할 뿐이고 언제든 끊을 수 있다.

### 4.5.9 지속 커넥션의 제한과 규칙
#### [지속 커넥션의 제한과 규칙](https://velog.io/@hustle-dev/HTTP-%EC%99%84%EB%B2%BD-%EA%B0%80%EC%9D%B4%EB%93%9C-3%EC%A3%BC%EC%B0%A8)
- 클라이언트가 해당 커넥션으로 추가적인 요청을 보내지 않을 것이라면 마지막 요청에 Connection: close 헤더를 보내고, 이를 보내면 그 커넥션으로 추가적인 요청을 보낼 수 없다.
- 커넥션에 있는 모든 메시지가 자신의 길이 정보를 정확히 가지고 있을 때에만 커넥션을 지속시킬 수 있다.(Content-length or 청크 전송 인코딩으로 인코드 되어 있어야함)
- HTTP/1.1의 프락시는 클라이언트와 서버 각각에 대한 별도의 지속 커넥션을 맺고 관리해야 한다.
- 커넥션 관련 기능에 대한 클라이언트의 지우너 범위를 알고 있지 않은 한 지속 커넥션을 맺으면 안된다.(오래된 프락시가 Connection 헤더를 전달할 수 있어서)
- HTTP/1.1 기기는 언제든 커넥션을 끊을 수 있다. (Connection 헤더의 값과 상관 없이)
- HTTP/1.1 애플리케이션은 중간에 끊어지는 커넥션을 복구할 수 있어야만 한다.
- 클라이언트는 전체 응답을 받기전 커넥션이 끊어지면 요청을 반복해서 보내도 문제가 없는 경우 요청을 다시 보낼 준비가 되어있어야한다.
- 하나의 사용자 클라이언트는 서버의 과부화를 방지하기 위해 넉넉잡아 2개의 지속 커넥션만 유지해야한다.

## 4.6 파이프라인 커넥션
HTTP/1.1에선 지속 커넥션을 통해 요청을 파이프라이닝 할 수 있다. 요청의 응답이 도착하기전에 큐에쌓여 바로바로 요청이 전달된다.

#### 파이프라인의 제약 사항
- 클라이언트는 지속 커넥션임을 확인하기 전까지 파이프라인을 이어서는 안됨.
- HTTP 응답은 요청 순서와 같게 와야한다.
- HTTP 클라이언트는 커넥션이 언제 끊어지더라도, 완료되지 않은 요청이 파이프라인에 있으면 언제든 다시 요청을 보낼 준비가 되어 있어야 한다. 이 말은 예상치 못하게 끊긴 커넥션을 다시 맺고 요청을 보낼 수 있어야한다는 말이다.
- POST 요청처럼 비멱등 요청(연산이 일어날때 서버의 데이터와 같이 결과에 변화가 생기는 것)은 에러가 발생하면 어떤 것들이 서버에서 처리되었는지 알 수 없기때문에 위험한 메서드의 요청은 보내선 안된다.

## 4.7 커넥션 끊기에 대한 미스터리
커넥션관리의 명확한 기준은 무엇일까.

### 4.7.1 '마음대로' 커넥션 끊기
마음대로 끊는다면 유휴시간에 요청메시지가 올 수 있음.

### 4.7.2 Content-Length와 Truncation
HTTP 응답은 정확한 크기값, Content-Length헤더를 가지고 있어야 한다. 이것을 기준으로  전달 받은 엔티티가 올바른지 확인해야한다.<br>
캐시 프락시가 수신자일경우 캐시하지않고 Content-Length를 정정해서도 안되고 받은 그대로 전달해야한다.

### 4.7.3 커넥션 끊기의 허용, 재시도, 멱등성
커넥션은 언제든지 문제가 없는 상황에서도 끊어질 수 있다. 적절히 재시도해야하는데 파이프라인과 같은 커넥션 상황에서 어떤것이 처리되었는지 아닌지 명확하게 알 수없기에 멱등을 고려해 안전하지않은 메서드는 자동으로 실행해선 안된다.

### 4.7.4 우아한 커넥션 끊기
TCP 커넥션은 양방향<br>
각 종단은 입출력 큐가 있다.

#### 전체끊기와 절반끊기.
close()는 입출력 큐 모드 끊는다. (전체 끊기)
shutdown()은 둘 중 어느것을 개별적으로 끊을 수 있다. (절반 끊기)

#### 리셋에러
여러 구성요소와 함께 통신할때, 파이프라인 지속 커넥션을 사용할 때 예상치 못한 쓰기 에러 방지를 위해 절반끊기를 사용해야한다.<br>
클라이언트에서 데이터를 보내지않을 것을 확신할 수 없는 이상 입력채널을 끊는것은 위험하다 따라서 통상 출력채널을 끊는게 안전하다. 반대편에선 모든 데이터를 읽고 데이터 전송이 끝남과 동시에 이쪽에서 커넥션을 끊음을 알게 될 것이다.<br>
파이프라인을 통해 10 개의 요청이 전송되고 일부는 응답받아 버퍼에 있지만 읽지 못한상황, 다음 요청을 보내고 서버는 너무 긴 시간임을 판단해 연결을 끊어버린다. 종료된 커넥션에 보낸 요청이기에 에러메세지를 받고 버퍼의 데이터를 지워버린다.<br>
이러한 상황은 잘 도착한 데이터도 리셋해버린다.

#### 우아하게 커넥션 끊기
1. 출력채널을 끊는다.
2. 다른쪽의 출력 채널이 끊기기를 기다린다.
3. 양쪽에서 서로 더는 데이터를 전송하지 않을 것이라고 알려주면(1번) 리셋의 위험없이 완전히 종료된다.

그렇지만 상대방이 절반끊기를 구현했는지는 미지수이기에 데이터스트림의 끝을 판별하기위해 주기적인 검사를 해야한다.
