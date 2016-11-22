작성 중...
http://vertx.io/docs/vertx-circuit-breaker/java/


### Vertx Circuit Breaker

- circuit breaker pattern을 구현하였다.
- 실패 횟수를 추적하며 임계치에 도달했을 때 circuit을 발동시킨다. 옵션으로 fallback을 실행한다.
- 실패하였을 경우 아래 세 가지를 지원한다.
 - failures reported
 - exception thrown
 - timeout
-  논블로킹/비동기로 구현된 cb가 운영을 보호해 vertx 모델의 이점을 누릴 수 있다.

#### Using the vert.x circuit breaker

#### Using the circuit breaker

#### Callbacks

#### Event bus notification

#### The half-open state

#### Using Netflix Hystrix

### etc

#### circuit overview

- circuit Breaker란, 원격 접속의 성공/실패를 카운트하여 에러율(failure rate)이 임계치를 넘어섰을 때 자동적으로 접속을 차단하는 시스템입니다. 
- circuit Breaker는 상태 머신(State Machine)으로 나타낼 수 있습니다. 접속 성공과 실패 이벤트가 발생할 때마다 내부 상태를 업데이트하여 자동적으로 장애를 검출하고 복구 여부를 판단합니다.
- 상태조건
 - CLOSED
  - 초기 상태입니다. 모든 접속은 평소와 같이 실행됩니다.
 - OPEN
  - 에러율이 임계치를 넘어서면 OPEN 상태가 됩니다. 모든 접속은 차단(fail fast)됩니다.
 - HALF_OPEN
  - OPEN 후 일정 시간이 지나면 HALF_OPEN 상태가 됩니다. 접속을 시도하여 성공하면 CLOSED, 실패하면 OPEN으로 되돌아갑니다.

 [![cb-status](https://github.com/leeplay/study/blob/master/image/cb-status.png?raw=true)]()

#### Armeria (line cb)
- Armeria는 LINE이 오픈소스로 공개한 Netty 기반의 비동기 Thrift 클라이언트/서버 라이브러리입니다.
- Armeria 0.13.0부터 Circuit Breaker를 decorator로 추가할 수 있게 되었습니다.

[![armeria-client-cb](https://github.com/leeplay/study/blob/master/image/armeria-client-cb.png?raw=true)]()

- grouping (설정지원)
 - Per Method
 - Per Host
 - Per Host and Method

[![armeria-cb](https://github.com/leeplay/study/blob/master/image/armeria-cb.png?raw=true)]()

- Failure Rate

[![armeria-failure-rate](https://github.com/leeplay/study/blob/master/image/armeria-failure-rate.png?raw=true)]()

- CB-Monitoring

- 더 자세히 알고 싶다면
 - http://developers.linecorp.com/blog/ko/?p=246#more-246
