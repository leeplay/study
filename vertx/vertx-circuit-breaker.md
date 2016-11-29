### Vertx Circuit Breaker

- circuit breaker pattern을 구현하였다.
- 실패 횟수를 추적하며 임계치에 도달했을 때 circuit을 발동시킨다. 옵션으로 fallback을 실행한다.
- 실패하였을 경우 아래 세 가지를 지원한다.
 - failures reported
 - exception thrown
 - timeout
-  논블로킹/비동기로 구현된 cb가 운영을 보호해 vertx 모델의 이점을 누릴 수 있다.


#### Using the circuit breaker

- cb를 생성하고 설정(타임아웃, 실패카운트)
- breaker 실행

```JAVA
CircuitBreaker breaker = CircuitBreaker.create("my-circuit-breaker", vertx,
    new CircuitBreakerOptions()
        .setMaxFailures(5) // number of failure before opening the circuit
        .setTimeout(2000) // consider a failure if the operation does not succeed in time
        .setFallbackOnFailure(true) // do we call the fallback on failure
        .setResetTimeout(10000) // time spent in open state before attempting to re-try
);

breaker.execute(future -> {
  // some code executing with the breaker
  // the code reports failures or success on the given future.
  // if this future is marked as failed, the breaker increased the
  // number of failures
}).setHandler(ar -> {
  // Get the operation result.
});
```

- executed block은 future 오브젝트를 파라미터로 받아 결과뿐만 아니라 조작의 성공 또는 실패를 표시합니다.
- 다음의 예는 Restful 엔드포인트를 호출하는 예제입니다.

```JAVA
CircuitBreaker breaker = CircuitBreaker.create("my-circuit-breaker", vertx,
    new CircuitBreakerOptions().setMaxFailures(5).setTimeout(2000)
);

breaker.<String>execute(future -> {
  vertx.createHttpClient().getNow(8080, "localhost", "/", response -> {
    if (response.statusCode() != 200) {
      future.fail("HTTP error");
    } else {
      response
          .exceptionHandler(future::fail)
          .bodyHandler(buffer -> {
            future.complete(buffer.toString());
          });
    }
  });
}).setHandler(ar -> {
  // Do something with the result
});
```

- 오퍼레이션의 결과는 아래 두 가지를 제공합니다. 
 - execute 메소드가 실행될 때 Future를 반환합니다.
 - executeAndReport 메소드가 실행될 때 Future를 제공합니다.

- 옵션으로 circuit이 오픈될 때 fallback을 실행할 수도 있습니다.
- fallback은 circuit이 오픈될 때 호출되거나 isFallbackOnFailure이 enable되어 있으면 호출됩니다.

 ```JAVA
 CircuitBreaker breaker = CircuitBreaker.create("my-circuit-breaker", vertx,
    new CircuitBreakerOptions().setMaxFailures(5).setTimeout(2000)
);

breaker.executeWithFallback(
    future -> {
      vertx.createHttpClient().getNow(8080, "localhost", "/", response -> {
        if (response.statusCode() != 200) {
          future.fail("HTTP error");
        } else {
          response
              .exceptionHandler(future::fail)
              .bodyHandler(buffer -> {
                future.complete(buffer.toString());
              });
        }
      });
    }, v -> {
      // Executed when the circuit is opened
      return "Hello";
    })
    .setHandler(ar -> {
      // Do something with the result
    });
 ```

 - fallback 함수를 설정할 수도 있습니다. fallback 함수는 Throwable을 파라미터로 취해 기대되는 타입으로 돌려줍니다.
 - fallback을 cb에 바로 설정하는 방법입니다.

 ```JAVA
 CircuitBreaker breaker = CircuitBreaker.create("my-circuit-breaker", vertx,
    new CircuitBreakerOptions().setMaxFailures(5).setTimeout(2000)
).fallback(v -> {
  // Executed when the circuit is opened.
  return "hello";
});
 ```

- 샘플코드

[![cb-sample](https://github.com/leeplay/study/blob/master/image/cb-sample.png?raw=true)]()

#### Callbacks

- circuit이 열리거나 닫혔을 때 callback을 설정할 수 있습니다.
- halfOpenHandler call을 통해 cb가 half-open 상태를 시도할 때 noty를 받을 수도 있습니다.



#### Event bus notification

- 매 순간 circuit의 상태는 변하고 각 이벤트는 event bus위에서 발송됩니다.
- setNotificationAddress를 설정해 event가 전송되는 주소를 설정할 수 있습니다.
- 만약 null이면 이 메소드는 그대로 통과되고 비활성화 됩니다. 디폴트로 vertx.circuit-breaker 주소를 사용합니다. 
- 각 이벤트 단위는 아래와같으며 json object를 전송합니다. 
 - state : OPEN, CLOSED, HALF_OPEN
 - name : cb의 이름
 - failures : 실패횟수
 - node : node의 식별자


#### The half-open state

- cb가 open 되면 실제 호출을 시도하지 않고 cb는 즉시 fail을 발생시킵니다.
- 적절한 시간이 흐르면(setResetTimeout) cb는 성공할 시도가 높다고 결정하여 half-open상태가 됩니다.
- half-open상태 다음 호출부터는 메소드의 실행을 허용합니다.
- call이 성공하면 cb는 closed 상태로 되돌아가고 정상처리를 위한 준비를 하게ㅗ딥니다.
- call이 실패하면 cb는 다음 타임아웃까지 open상태를 돌아갑니다.

#### Using Netflix Hystrix

- Hystrix는 cb 패턴을 구현하였습니다.
- vertx 어플리케이션에서 hystrix를 vertx-circuit breaker 대신 사용할 수 있습니다.
- command의 실행은 blocking되므로 worker verticle이나 executeBlocking 안에서 실행해야 합니다.
- Hystrix의 비동기 지원을 사용하는 경우 vert.x 스레드에서 콜백이 호출되지 않고 실행 전에 컨텍스트에 대한 참조(getOrCreateContext)를 유지해야합니다.
- 그리고 이 콜백에서 runOnContext를 사용하여 이벤트 루프로 다시 전환하십시오. 이 기능이 없으면 Vert.x 동시성 모델을 잃어 버리고 동기화와 오더링을 직접 관리해야 합니다. (Hystrix는 못 쓰겠다;;;)

```JAVA
vertx.runOnContext(v -> {
Context context = vertx.getOrCreateContext();
HystrixCommand<String> command = getSomeCommandInstance();
command.observe().subscribe(result -> {
context.runOnContext(v2 -> {
// Back on context (event loop or worker)
String r = result;
});
});
});
```

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
