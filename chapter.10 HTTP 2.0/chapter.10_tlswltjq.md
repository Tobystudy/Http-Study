# HTTP/2.0
## 등장배경
HTTP/1.1의 메시지 포맷은 단순성과 접근성에 목표를 두고 최적화 하였음
따라서 성능은 희생대상.

커넥션 하나를 통해 요청 하나, 응답 하나를 받는 메시지 교환 방식은 단순해서 좋았지만 응답을 받아야만 다음 요청을 보낼 수 있어 회전 지연을 피할 수 없었다.
문제를 회피하기위해 병렬 커넥션, 파이프라인 커넥션 등이 도입 되었으나 근본 성능문제는 해결할 수 없다.

#### SPDY
헤더를 압축해 대역폭을 절약함
TCP커넥션 하나에 여러 요청을 동시에 보내기 가능
HTTP2.0은 SPDY를 기반으로 설계되었음

## HTTP/1.1과의 차이
### 프레임
HTTP/2.0에서는 모든 메시지가 프레임에 담겨 전송된다.

| 필드 이름           | 설명                                                                                         |
|---------------------|----------------------------------------------------------------------------------------------|
| R (Reserved)        | 현재는 사용되지 않으며, 항상 0으로 설정됩니다. 예약된 비트입니다.                               |
| 길이 (Length)       | 프레임 페이로드의 길이를 나타냅니다. 최대 16,777,215바이트까지의 길이를 표현할 수 있습니다.    |
| 종류 (Type)         | 프레임의 유형을 나타냅니다. 데이터 프레임, 헤더 프레임, 우선 순위 프레임 등이 있습니다.         |
| 플래그 (Flags)      | 프레임에 대한 추가 제어 정보를 포함합니다. END_STREAM, END_HEADERS 등의 플래그가 있습니다.     |
| R (Reserved)        | 두 번째 예약된 비트입니다. 현재는 사용되지 않으며, 항상 0으로 설정됩니다.                       |
| 스트림 식별자      | 프레임이 속한 스트림을 식별하는 데 사용됩니다. 해당 스트림 식별자가 없는 경우, 0x0으로 설정됩니다. |
### 스트림과 멀티플렉싱
>스트림은 HTTP/2.0 커넥션을 통해 교환되는 프레임들의 양방향 시퀀스

1.1에서는 하나의 커넥션이 열리면 요청에 대한 응답이 도착하지 않으면 같은 커넥션으로 다시 요청할 수 없었지만 2.0에서는 하나의 커넥션이 열리고 여러개의 스트림이 동시에 열릴 수 있다.
- 스트림은 우선순위를 가질 수 있다.
- 모든 스트림은 고유한 식별자를 갖는다.
- 스트림 식별자는 다시 사용할 수 없다. 고갈되면 새 커넥션 필요
### 헤더압축
HTTP/1.1은 압축없이 헤더 전송 과거에 비해 매우 많은 요청덕에 헤더의 크기는 문제가 됨
이를 개선하기위해 HTTP/2.0은 헤더를 압축해 헤더 블록 조각으로 쪼개 전송한다.
### 서버 푸시
HTTP/2.0은 서버가 하나의 요청에 대해 여러개의 리소스를 응답으로 보낼 수 있다.
서버가 클라이언트가 어떤 리소스를 요구할 것인지 알 수 있는 상황에서 유효하다. ex)웹페이지의 경우 css, 이미지 파일등을 요구할 예정인것은 자명하다.
## 알려진 이슈
### 중개자 캡슐화 공격
HTTP/2.0 -> HTTP/1.1메시지로 변환할 때 메시지의 의미가 변질 될 가능성이 있다.
### 긴 커넥션 유지로 인한 개인정보 누출 우려
HTTP/2.0은 사용자 요청을 보낼 때 회전 지연을 줄이기 위해 클라이언트와 서버 사이의 커넥션을 오래 유지하는 것을 염두에 두고있는데 이를 악용할 우려가 있다.
HTTP가 가진 문제 그 자체 이지만 짧게 유지되는 커넥션에 비해 위험이 크다.
