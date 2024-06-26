# http 2.0?
## 정의와 도입 배경
- Http의 최초 등장 배경은 기본적으로 구현의 단순성과 접근성이 가장 큰 목적
- 하지만 성능과 비효율성은 고질적인 문제였음
- http 1.1의 등장 배경도 지연 커넥션의 공식 스펙화가 가장 큰 축
- 그럼에도 Request/Response 응답쌍에 대한 동기적 수신 지연문제는 여전해서 병렬 커넥션 / 파이프 라이닝등으로 해결하려 노력
  - 병렬 커넥션은 커넥션을 맺는 행위 자체의 오버헤드가 존재
  - 파이프라이닝은 HOLB 문제
- 이를 해결하기 위해 다양한 시도가 존재

### 구글의 SPDY
- 그중 가장 큰 성공을 거둔 아키텍처는 구글의 SPDY
- 이를 베이스로 http의 새로운 표준 스펙을 규정하자는 움직임이 있었고 http 2.0이 나오게됨

### 파이프라이닝과 HOLB
- 지속 커넥션을 극단적으로 사용하는 방식으로 요청에 순서를 메기고 하나의 커넥션에 순차적으로 요청을 보낸 뒤, 응답도 해당 순서대로 받음으로써 rtt 지연을 줄이는 방식
- 당연한 문제지만 요청의 순서대로 응답도 순서를 지켜야하므로 특정 순서의 요청 응답이 밀리면 다 밀린다.
- 순서를 지키는 방식이 아니라 요청 키와 응답을 매핑하면 안됬을까? -> http 2.0의 스트림 번호 해결법
- 실제 파이프라이닝 기술은 거의 사용되지 않고 사장됨, 대부분의 브라우저들도 비지원한다고함

## 바이너리 프레이밍? 프레임 헤더?
- http 2.0 은 바이너리 프레이밍으로 더 가볍다고 함
- 어차피 모든 요청응답은 결국 직렬화와 인코딩을 통해 바이너리로 보내지고 받는데...?
- 헤깔리는 용어인데, 결국은 IP, TCP 처럼 아키텍처를 바이너리 레벨의 프레임구조로 갖는다는 의미

![image](https://github.com/Tobystudy/Http-Study/assets/85499582/6e82910d-b4b3-45b6-822d-ccd1d2ad5b86)

- 위의 TCP 처럼 몇번째 비트값이 있으면 뭐가 있다 이런식으로 좀더 구조화된 프로토콜이란 의미

이를 통한 장점은 기존 HTTP 1.1까진 결국 컴퓨터가 이해할 수 없는 텍스트의 나열이므로 content-lengh 길이의 끝까지, crfd 를 받을때까지 모든 메시지를 다 받은 후 파싱이 가능했으며, 반드시 key-value 쌍으로 보냈어야했으나
약속된부분에 대한 키를 명시할 필요가 없어지고, 프레이밍 단위로 끊어서 파싱이 가능하므로 컴퓨터가 파싱하고 이해하는 청킹 단위를 더 작게 가져갈 수 있게됨

- 반대로 각 프레임 마다 프레임 헤더가 붙는 약간의 오버헤드도 존재함!

<img width="1465" alt="image" src="https://github.com/Tobystudy/Http-Study/assets/85499582/33ac6767-fb6f-4fd8-b588-677569cb7662">

- 실제로 와이어샤크로 캡쳐해보면 하나의 tcp 패킷안에 두개의 http2 프레임이 존재함을 확인가능
  - 각각은 http2.0상의 헤더와 메시지이며 [5]는 스트림 번호

## 스트림과 멀티 플렉싱

- 기존 요청 / 응답의 아키텍처가 아닌 반드시 묶여야할 논리적 요청 흐름을 스트림으로 규정
- 따라서 애플리케이션 레이어의 HOLB 문제가 생기지 않음, 스트림번호만 잘지키면 요청에 다른 응답이 다른 순서로 와도 문제가 없다
- 멀티플렉싱이 가능해진셈!

<img width="822" alt="image" src="https://github.com/Tobystudy/Http-Study/assets/85499582/0a4533ff-41e1-43a5-ac2b-2803f5c3728b">


## 서버 푸시

- 서버가 해당 요청 이후 필요할것으로 예상되는 요청을 미리 정의해 보내주는것
- NEXT가 하는 프리페칭이랑 비슷한듯한 방식
- 리소스를 푸시하려는 서버는 PUSH_PROMISE 프레임을 보내 미리 알려주고 클라이언트는 이 응답을 받을 예약을 하거나, RST_STREAM 프레임으로 거절
- 서버는 오직 안전하고 캐시가능하고 본문을 포함하지 않은 요청에 대해서만 푸시해야함
- 동일 출처 정책을 지켜야함(CORS)

## 헤더 압축
- 모든 요청 응답에 헤더를 꽉꽉 눌러담아야하는 기존방식에 비해 http2.0은 중간 캐싱 레이어가 존재해서 헤더를 캐싱해둘수 있음
<img width="859" alt="image" src="https://github.com/Tobystudy/Http-Study/assets/85499582/d83b6385-abad-4ba4-99fe-0fc1d946bc49">

## 알려진 보안 이슈
### 중개자 캡슐화 공격
- 중개자가 간섭할수있다면 언제든 발생하는 문제아닐까? http 1.1도 동일한 문제는 발생할 수 있다고 봄
- 단지 암호화의 문제일 뿐이다

### 긴 지연 커넥션문제
- 이역시 http 자체의 문제

## GRPC
- 구글에서 개발한 고성능, 오픈소스, 다중 언어 지원 RPC 프레임워크
- 기반기술로 http 2.0을 사용한다!
- HTTP 2.0은 자연어로 된 부분이 더 적어졌으므로(프레이밍등으로 보다 컴퓨터에 친화적) 이를 해석해줄 중간 레이어 / 도구가 필요함
- grpc는 IDL을 통해 이를 자연어로 보여주고 클라이언트 서버를 프레임워크차원으로 제공해줌!

### GRPC 통신 방식
#### Unary : Request / Response
- 기존 http 와 동일

#### Client / Server Stream

- 단방향 스트림

#### Bidirect Stream
- 양방향 스트림

### Proto Buff
구조화된 데이터를 직렬화하기위한 언어/플랫폼 중립적 확장 가능한 메커니즘

#### 장점
- 이진 데이터 포맷 사용 (데이터 사이즈↓, 직렬화 속도↑)
- 다양한 언어지원 (protoc 컴파일러로 다양한 언어 코드 생성)
- 버전 호환성 (필드 번호 기반 데이터 구조)
- 타입 안정성 (명시적 타입 선언)

#### 단점
- 이진 데이터 포맷 (가독성↓)
- 러닝커브


