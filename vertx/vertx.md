
#### Reactor 개요

vertx1
[![vertx1](https://github.com/leeplay/study/blob/master/image/vertx1.jpg?raw=true)]()


- 이벤트에 반응하는 reactor를 만들고 reactor에 이벤트를 처리할 event handler들을 등록한다. (initiate)
- reactor는 이벤트가 발생하기를 기다린다. (receive)
- 이벤트가 발생하면 이벤트를 처리할 event handler단위로 분할한다.(demultiplex)
- 분할된 이벤트를 해당 event handler에게 발송한다. (dispatch)
- event handler에 알맞은 method를 사용하여 이벤트를 처리한다. (process event)


#### Vertx 개요

- 2011년 팀폭스 개발
- Eclpipse Foundation 에서 관리
- Node.js에서 영감을 얻어 시작
- 신기술은 아니며 I/O Demultiplexer에 기초한 Reactor 모델
- 다중 스레드를 사용하지 않고 단일 스레드 상에서 많은 작업을 동시에 수행할 수 있는 게 Reactor 동시 처리 모델

#### Vertx Reactor 

- Initation Dispatcher에서는 이벤트 핸들러의 Concrete Event Handler를 등록, 해제, 호출하기 위한 인터페이스가 있어 블록된 상태를 유지하다가 Demultiplexer를 통해 처리해야 할 한 개 이상의 이벤트를 감지하면 각 이벤트에 대응되는 이벤트 핸들러를 차례로 호출한다. 
- Initation Dispatcher는 단일 스레드에서 동작하므로 이벤트 핸들러의 동시 샐행을 걱정할 필요는 없다. 
- Initation Dispatcher와 이벤트 핸들러의 느슨한 관계는 개발자가 집중해야 할 비즈니스 로직 부분을 이벤트 핸들링 부분과 분리해주어 Event Driven 상황이 필요한 곳에서 Reactor 동시 처리 모델을 쉽게 적용할 수 있다. 
- Initation Dispatcher는 단일 스레드에서 동작하므로 동기화 문제에서 벗어날 수 있지만 본질적으로 정교한 시분할 기법으로 각각의 작업을 순차적으로 처리하는 것이다.
- 블록될 가능성이 있는 API를 호출하거나 이번트 처리가 지연된다면 이는 곳 애플리케이션 전체에 심각한 병목을 가져오믐을 의미한다. 또한 여러 개의 CPU를 가지고 이용할 수 있는 병렬 처리에 대한 이점을 얻을 수 없다는 단점이 있다. 
- 실제 이러한 단점은 Reacotr 동시 처리 모델에 기반을 둔 Vertx나 Node에서 그대로 존재하지만 이러한 단점을 보완하기 위한 구조를 가지고 있다. 이러한 구조를 MultiReactor라고 한다. 
- 이벤트 기반 비동기 어플리케이션을 개발할 때 반드시 지켜야 할 규칙은 Initiation Dispatcher가 실행되는 스레드를 블록시키거나 시간이 오래 걸리는 작업을 실행하느라 이벤트 처리가 지연되지 않게끔 해야 한다는 것이다. 
- Reactor 동시 처리 모델은 연결되는 TCP 채널의 개수와 관계없이 항상 일정한 스레드 개수가 유지되어 불필요한 오버헤드가 없다. 
- 일반적인 Reactor 동시 처리 모델은 단일 스레드에서 동작하여 병렬 처리의 이점을 얻을 수 없다는 단점이 있지만 Vertx에서는 이러한 단점을 극복했다. 

#### Vertx Multireactor

- 시스템의 CPU 개수만큼 이벤트 루프 스레드를 생성해서 이벤트를 처리하는데 각각의 이벤트 루프 스레드는 서로 다른 클래스로더로 만들어진 완전히 독립적인 코드를 실행하므로 동기화 문제는 발생하지 않는다. 
- 이렇게 특정 이벤트 루프 스레드에서 한번 실행된 코드는 절대 다른 이벤트 루프 스레드에서 실행되지 않음을 보장한다.

[![vertx2](https://github.com/leeplay/study/blob/master/image/vertx2.jpg?raw=true)]()

- 이벤트 루프 쓰레드에서 실행되는 코드 단위를 Verticle이라고 한다.
- 각 이벤트 루프 스레드는 서로 독립적인 Verticle을 실행한다.
- 각 Verticle은 서로 다른 클래스로더에서 만들어지므로 A라는 Verticle이 이벤트 루프 스레드1과 2에서 동시에 실행되고 있어도 독립된 상태를 유지한다. 
- 즉 Verticle 내 static 으로 선언한 변수를 포함한 어떤 값도 공유되지 않는다.
- Verticle은 최초 할당된 이벤트 루프 스레드에서만 실행됨을 보장하며 그 외 이벤트 루프 스레드에서는 절대 실행되지 않는다. 

#### Vertx 주요 특징

- 단순함
- 다양한 언어지원 
 - Java, JavaScript, Ruby, Python, Groovy, Clojure, Scala
 - 2개 이상의 언어 동시 사용 가능
- JVM
 - JVM위에서 동작
 - JVM의 안전성, 성능의 이점을 가짐
- 범용성
 - 웹어플리케이션, P2P, 로드밸런서, 프락시 서버 등 이벤트 주도가 필요한 상황에 적용
- 확장성
 - 네트워크 상에서 서로 다른 JVM에서 동작하는 Vetx를 클러스터로 구성가능 
 - 동일한 클러스터에 묶인 Vertx끼리 이벤트 버스를 통해 String, JSON, Byte Buffer 형태의 메시지를 주고 받을 수 있음
 - Vertx에서 제공하는 SockJS Event Bus Bridge를 사용하면 클라이언트 사이드의 웹브라우저까지도 클러스터에 참여시킬 수 있음
- 내부 컴포넌트 구성
 - Vert.x를 만든 사람들이 대단하다고 느끼는 점은, 다 처음부터 개발한 것이 아니라 대부분의 모듈을 기존의 오픈소스들을 기반으로 해서 개발하였다는 것이다.
 - 고속 네트워크 처리를 위해서는 국내 개발자 이희승씨가 만들어서 더 유명한 Apache Netty를 서버 엔진으로 사용하고 있고, Vert.x 노드간의 통신을 지원하기 위해서, 데이타그리드 솔루션인 HazelCast를 사용하고 있다.
- 클러스터링 지원
 - Vert.x는 클러스터링 지원이 가능하다. 여러개의 Vert.x 서버(instance)를 띄워서 구동이 가능하고, Vert.x 서버간에 통신이 가능하다. (메세지를 보낼 수 있다.)
 - Vert.x의 기능을 사용하지 않더라도, 내장되 HazelCast 기능을 사용하면, 인스턴스간의 데이타 공유 구현은 가능하다.
 - 참고로, Vert.x에 embedded된, HazelCast는 Community 버전이다. (무료 버전). HazelCast는 자바 기반이기 때문에, 대용량 메모리를 사용하게 되면, Full GC가 발생할때, 시스템의 순간적인 멈춤 현상이 발생하기 때문에, 이를 감안해서 사용하거나, 또는 상용 버전을 사용하면 Direct Memory라는 개념을 사용하는데, 이는 Java Heap을 사용하지 않고, Native 메모리를 바로 접근 및 관리 함으로써, 대용량 메모리를 GC Time이 없이 사용할 수 있다. HazelCast도 기능상으로 보면 대단히 좋은 솔루션이기 때문에, 한번 살펴보기를 권장한다
- 멀티 인스턴스 지원을 통한 Node.JS보다 빠른 성능
Vert.x의 ELP는 Single Thread에서 동작하지만, 동시에 여러개의 ELP를 띄울 수 있다. Multi Thread를 띄워서, 동시에 여러개의 Verticle을 실행할 수 있다는 이야기다. 자칫하면 WAS의 Multi threading 모델과 헷갈릴 수 있는데, 여러개의 Thread를 띄우더라도, 각 Thread는 독립적은 ELP를 가지고 동작하고, WAS의 MultiThread 모델과는 다르게, 각 Thread간에 객체 공유나 자원의 공유가 없이 전혀 다르게 독립적으로 동작한다.
이렇게 하나의 Instance에서 여러개의 Thread를 띄울 수 있기 때문에, Multi Core Machine에서는 좋은 성능을 낼 수 있다. Node.js의 경우 Single Thread기반의 ELP를 하나만 띄울 수 있기 때문에, Core수가 많아서, 이를 사용할 Thread가 없다. 그래서 Core가 많은 Machine에서 성능이 크게 늘어나지 않는 반면에, Vert.x는 동시에 여러개의 Thread에서 여러개의 ELP를 수행하기 때문에 성능이 더 높게 나온다. 물론 Node.js도 여러개의 Process를 동시에 띄워서 여러개의 Core를 동시에 사용할 수 는 있지만 Process의 Context Switching 비용이 Thread의 Context Switching 비용보다 크기 때문에, 여러개의 Thread 기반으로 동작하는 Vert.x가 성능에서 유리할 수 밖에 없다.
- Embedded Vertx
 - Vert.x는 자체가 서버로써 독립적으로 동작할 수 있을 뿐만 아니라 라이브러리 형태로도 사용이 가능하다. 즉 Tomcat같은 WAS에 붙어서 기동이 될 수 있다. 하나의 JVM에서 Tomcat 서비스와 Vert.x 서비스를 같이 수행하는 것이 가능하다. 예를 들어서 일반적인 HTTP Request는 Tomcat으로 처리하고, Socket.IO나 WebSocket과 같은 Concurrent connection이 많이 필요한 Request는 Vert.x 모듈을 이용해서 처리하게할 수 있다.

#### Vertx 용어

- Vert.x instance
 - Vert.x instance는 하나의 Vert.x 서버 프로세스로 보면 된다. 하나의 Vert.x JVM 프로세스를 하나의 Vert.x instance라고 이해하면 된다.

- 일반 Verticle (aka. standard verticle, ELP verticle)
 - 먼저 Verticle이다. Verticle은 Vert.x에서 수행되는 하나의 프로그램을 이야기 한다. 앞서 설명했듯이, 자바의 Servlet과 같은 개념으로 이해 하면 된다.

- Verticle instance
 - Verticle 코드가 로딩되서 객체화 되면, 이를 Verticle instance(하나의 Verticle Object)라고 한다.
 - Verticle은 ELP안에서 수행이 되며, 항상 같은 쓰레드에서 수행이 된다. Verticle instance는 절대 multi thread로 동작하지 않고, single thread에서만 동작한다

- Worker Verticle
 - 일반 Verticle은 개별 socket에 대해서 ELP를 돌면서 Event 처리를 한다. 그래서 해당 Event 처리에 오랜 시간이 걸리면 전체적으로 많은 성능이 떨어지기 때문에, 문제가 된다.
 - 예를 들어 하나의 event 처리에 100ms 가 걸리는 작업이 있다면, 연결된 Connection이 100개가 있을 경우 전체 socket에 대해서 한번 이벤트 처리를 하는데 10초가 걸리고, 처리가된 소켓이 다음 이벤트를 받을 수 있을 때 까지 10초가 걸린다. 즉 클라이언트 입장에서 연속적으로 요청을 받았을때, 두번째 요청에 대해서 응답을 받는 것은 10초후라는 이야기가 된다. 이는 Single Thread 모델이기 때문에 발생하는 문제인데, Vert.x에서는 이런 문제를 해결하기 위해서 Worker Verticle 이라는 형태의 Verticle을 제공한다.
 - Worker Verticle은 쉽게 생각하면, Message Queue를 Listen하는 message subscriber라고 생각하면 된다. Vert.x 내부의 event bus를 이용해서 메세지를 보내면, 뒷단의 Worker Verticle이 message Queue에서 메세지를 받아서 처리를 한 후에 그 결과 값을 다시 event bus를 통해서 caller에게 보내는 형태이다. (Asynchronous call back 패턴)
 - event bus로 request를 보낸 Verticle은 response를 기다리지 않고, 바로 다음 로직을 진행하다가, Worker Verticle에서 작업이 끝난 이벤트 메세지가 오면 다음 ELP가 돌때 그 이벤트를 받아서 응답 메세지 처리를 한다.
 - DB 작업이나, 시간이 오래 걸리는 작업은 이렇게 Worker Verticle을 이용해서 구현할 수 있다.

 [![vertx4](https://github.com/leeplay/study/blob/master/image/vertx4.jpg?raw=true)]()

- Worker Verticle instance & Thread pooling
 - 이 Worker Verticle의 재미있는 점은 Thread Pool을 지원한다는 것이다. ELP Verticle과 마찬가지로, Worker Verticle instance는 독립된 클래스 로드로 로딩되어 생성된 객체로 쓰레드에서 각각 다른 독립된 instance가 수행되기 때문에, Thread safe하다.
 - 단 Worker Thread의 경우 공용 Thread Pool에서 수행된다. ELP Thread의 경우 Verticle은 항상 지정된 Thread에서 수행되는데 반해, Worker Verticle Instance는 Thread Pool의 아무 쓰레드나 유휴 쓰레드에서 수행이 된다.

- Event Bus
 - Event Bus는 일종의 Message Queue와 같은 개념으로, Verticle 간에 통신이나, Vert.x Instance간의 통신이 가능하게 한다.
 - Verticle간의 통신이란 Java로 만든 Verticle에서 Python으로 만든 Verticle로 메세지 전달이 가능하다. 또한 Event Bus를 이용하면 서로 분리되어 있는 (다른 JVM 프로세스로 기동되는 또는 서로 다른 하드웨어에서 기동되고 있는) Verticle 간에 메세지를 주고 받을 수 있다.
 - 이 메세지 통신은 1:1 (Peer to Peer) 통신 뿐만 아니라, 1:N (Publish & Subscribe) 형태의 통신까지 함께 지원한다.
 - 타 일반적인 Message Queue와 다른 특성은 일반적인 Message Queue의 경우 Fire & Forget 형태의 Message exchange pattern을 사용하는데 반해서, Event Bus는 Call back pattern을 사용한다. 
 - 즉, JMS나 RabbitMQ의 일반적인 경우 클라이언트가 메세지 큐에 메세지를 넣고 바로 리턴하는데 반해서, Event Bus를 이용하면, 클라이언트가 메세지를 넣고 바로 리턴이 된 다음, 큐를 사용하는 Consumer가 메세지를 처리한 후에, 메세지를 보낸 클라이언트에게 다시 메세지를 보내서, 처리를 하도록 할 수 있다.
 - 예를 들어 하나의 HTTP request가 들어왔을때, event bus에 메세지를 넣고 해당 클라이언트는 내부적으로 리턴이 된다.(HTTP response는 보내지 않고, Connection은 물고 있다.) 그 후에 Worker Verticle에서 작업을 처리한 후 Call back rely를 보내면, 클라이언트에서 이 이벤트를 받아서 HTTP connection으로 response를 보낼 수 있다.
 - 이 구조가 있기 때문에, Single Thread모델임에도 불구하고 뒷단에서 비동기 메세지를 처리하는 방식을 이용하여 long transaction도 핸들링할 수 있게 된다.

- Shared data 처리
 - 하나의 Vert.x instance안에서만 공유 가능하다. (다른 Vert.x 인스턴스간에는 아직까지는 불가능)

- Module
 - 모듈은 하나의 Runnable Application으로, 자바의 일종의 WAR 파일이라고 생각하면 된다.
 - mod.json (WAR 파일의 web.xml 과 같은 메타 정보 description) 에 메타 정보를 정의한 후에, 클래스와 jar 파일등의 라이브러리, 그리고 기타 애플리케이션에서 사용할 파일들을 같이 묶어서 패키징 한 형태이다.
 - 이 Module은 Maven repository 시스템등에 저장 및 배포될 수 있다. (vagrant나 docker 컨셉과 비슷한듯. 이제 Runtime application도 repository를 사용하는 추세인가 보다)

- Pumping
 - 네트워크 Flow Control을 해주는 기능으로,  읽는게 쓰는거 보다 빠르면(예를 들어 socket에서 읽고, 파일에 쓰는 ) Queuing이 많이되고, 메모리 소모가 많아져서 exhausted된다. 이를 막이 위해서 쓰다가 Q가 차면 socket read를 멈추고 있다가 write q가 여유가 생기면 다시 읽는 것과 같은 flow control이 필요한데, 이를 pump라고 한다. 

- HA (High Availibility)
 - Vert.x는 HA 개념을 지원한다. Vert.x에서 HA란, HA mode로 Vert.x instance를 구성했을때, 해당 Vert.x instance가 비정상 종료 (kill등) 되면, 해당 instance에서 돌고 있던 Vert.x Module을 다른 Vert.x Instance로 자동으로 옮겨서 실행해주는 기능이다.
 - 실행시 -ha 옵션을 줘서 실행하는데, 하나의 클러스터 내에서도 hagroup을 지정하여 그룹핑이 가능하다. 예를 들어 클러스터에 10개의 노드가 있을때, 4개는 업무 A용 HA그룹으로 나누고, 6개는 업무 B용으로 나누는 식으로 업무별로 나눌 수 도 있고 Machine이 3개가 있을고, 2개의 업무를 수행하는데, 각 Machine마다 Vert.x instance를 2개씩 돌린다고 했을때 이런식으로 구성을 하게 되면, Machine 단위로 HA가 넘어가도록 구성을 할 수 있다.

```
Machine 1: vertx.instance=hagroup_A , vertx.instance2=hagroup_B 
Machine 2: vertx.instance=hagroup_A , vertx.instance2=hagroup_B 
Machine 3: vertx.instance=hagroup_A , vertx.instance2=hagroup_B 
```

- Clustering
 - HA 없이 클러스터링 구성만을 할 수 있는데, event_bus로 메세지를 주고 받을려면, 해당 vert.x instance들이 같은 cluster내에 들어 있어야 한다.
 - 설정은 conf/cluster.xml 을 이용해서 하는데, 이 파일은 hazelcast의 클러스터링 설정 파일이다. Default로는 multicast를 이용해서 cluster의 멤버를 찾도록 하는데, 인프라 특성상 multicast가 안되는 경우에는 (아마존과 같은 클라우드는 멀티캐스트를 지원하지 않는 경우가 많음). 직접 클러스터 노드들의 TCP-IP 주소를 적어두도록 한다.
 - 또한 서버의 NIC가 여러개인 경우에는 어떤 NIC를 사용할지 IP주소로 명시적으로 정의해줘야 한다. (유선,무선랜이 있는 경우 등). 직접 주소를 정해주지 않는 경우, HazelCast의 경우 NIC리스트에 있는 순서중 첫번째 있는 주소를 클러스터 주소로 사용한다.

#### Vertx 기타 
- vertex javascript socket.io 자바스크립트를 사용하면 모든 브라우저가 지원함, 모바일까지
- 브라우저가 판단해 아래 네 가지 중 하나를 선택해 지원함
  - 웹소켓 -> 플래쉬소켓 -> xhr폴링 -> json폴링
- verticle은 webapplication에서 스프링 빈의 생명주기를 따르도록 할 수 있다. 


#### Vertx 참고

Single Threaded Model이라서 더 빠르다? No.!!

Vert.x나 Node.JS와 같은 Single Threaded 모델의 서버들이 내세우는 장점이, Single Thread Model이기 때문에, Context Switching이 없고, 이로 인해서 성능이 더 빠르다. 라는 논리인데, 사실 DB나 File IO가 있을 경우, 뒷단의 Thread Pool에 request & call back 형식으로 처리를 하고 있기 때문에, 이 경우에는 Single Thread Model이 아니라 Multi Thread Model이 되고, 이 Thread Pool들의 Thread로 인한 Context Switching이 발생하게 된다.

이런 IO Call이 없는 경우 (Thread Pool로 작업을 보내지 않는 경우), 당연히 Context Switching 부하가 없기 때문에 빠르겠지만, DB 작업이 들어 갈경우,  http://www.techempower.com/benchmarks/#section=data-r8&hw=i7&test=query
의 자료를 보면, 오히려 Servlet이 더 빠른걸 볼 수 있다.

servlet-raw : resin + mysql
nodejs-mysql : nodejs + mysql

[![vertx3](https://github.com/leeplay/study/blob/master/image/vertx3.jpg?raw=true)]()

#### Vertx 구동 순서

#### Starter
- default 동작과  -cp를 주었을 때 처리 방식은 다르지만   run 을 호출함
- runbare와 runVerticle 두가지 모드가 있음 
- runBare 는 현재 구현 중인듯 ? run대신 -ha를 사용하는 해당 명령어가 메뉴얼에 없음
- runVerticle
     - ha, cluster 옵션을 확인함
     - startVertx를 호출함
       - VertxMetricFactory를 생성함 -> 실질 구현체 없음
       - vertx로 시작하는 환경변수들을 찾음
       - vertx 시작전 할 일 함 -> 실질 구현체 없음
       - VertxFactory가 생성됨
         - 프로퍼티 초기화
         - cluster 로 생성할지 확인
         - VertxFactory를 로드 후 Vertx 를 생성함
            - checker : BlockedThreadChecker
            - eventLoopGroup : NioEventLoopGroup
            - workerPool : newFixedThreadPool
             - internalBlockingPool : newFixedThreadPool
             - workerOrderedFact : OrderedExecutorFacotry
             - internerOrderedFact : OrderedExecutorFacotry
             - FfileResolver : FileResolver
             - deployManager : DeployManager
             - initialiseMetrics : 실질 구현체 없음, VertxMetricsFactory
             - cluster 옵션이 켜져있으면 관련작업함 
             - shareData : SharedDataImpl
             - eventBus : EventBusImpl
      
     - -instances수를 확인함, 없음 1
     - -conf 확인
     - -cp 옵션확인 없음 . 만 확인
     - deployOption 생성

#### 추가설명

- Factory는 BufferFactory, FUtureFactory, PumpFactory, VertxFactory, WebSocketFrameFacotry를 meta-inf/service에 선언해둠 
- BlockedThreadChecker는 VertexThread Map에 등록된 Thread의 수행시간을 계산해 blocked exception을 발생시킨다. 나노타임 기준임, 아래 생성되는 모든 쓰레드를 감시함
 - 이벤트 루프 수행시간 : 2000000000
 - 워커수행시간 : 60000000000
 - 경고시간 : 5000000000


- NioEventLoopGroup
 - nThreads : core * 2
 - ThreadFactory
 - vert.x-eventloop-thread 를 core *2개만큼 생성한다.
 - 이벤트 처리를 위한 루프(SingleThreadEventExecutor)가 16개가 생성된다. 
 - EventExcutor 를 선택하는 방식도 결정한다.  항상 순서를 보장하는 방식임
 - 16개의 쓰레드는 MultithreadEventExecutorGroup 로 관리된다.


- workerPool
 - vert.x-worker-thread- 이름으로 풀 생성
 - 풀의 디폴트 사이즈는 20
 - task를 가져와서 할당해서 순서를 보장

- internalBlockingPool
 - vert.x-internal-blocking- 이름으로 풀 생성
 - 풀의 디폴트 사이즈는 20
 - task를 가져와서 할당해서 순서를 보장


- fileResolver 
 .vertx/file-cache-3d3339a0-4758-4ec9-9716-1ec75fbf010f 캐시용 디렉터리 생서함


- DeploymentManager
 - meta-file로 정의되어 있지 않기 때문에 serviceLodader는 수행되지 않음
 - List<VerticleFactory> defaultFactories 생성, JavaVerticleFactory를 담음

- EventBusImpl
 - BufferFactory를 생성함
 - EventLoopContext 생성함

#### APIGW에서 봐야될 소스
 http://kimseunghyun76.tistory.com/406
 http://vertx.io/docs/vertx-core/java/
 http://vertx.io/docs/vertx-web/java/
 http://vertx.io/docs/vertx-hazelcast/java/
 http://vertx.io/docs/vertx-rx/java/
 http://vertx.io/docs/vertx-reactive-streams/java/
 http://vertx.io/docs/vertx-tcp-eventbus-bridge/java/
 http://vertx.io/docs/vertx-sync/java/
