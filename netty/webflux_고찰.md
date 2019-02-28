### WebFlux 를 알기 전에 ...

- non-blocking
- Netty, Undertow, and Servlet 3.1+ containers


#### blocking vs non-blocking

```java
private void run() throws IOException {
    ServerSocket server = new ServerSocket(8888);
    System.out.println("접속 대기중");

    while (true) {
        Socket sock = server.accept();
        System.out.println("클라이언트 연결됨");

        OutputStream out = sock.getOutputStream();
        InputStream in = sock.getInputStream();

        while (true) {
            try {
                int request = in.read();
                out.write(request);
            }
            catch (IOException e) {
                break;
            }
        }
    }
}
```

- 서버 소켓의 accept가 병목
- 블로킹 모드이기 때문에 여러 클라이언트가 동시에 접속 요청을 하는 상황에서 대기시간이 길어김
- 클라이언트 수가 증가하면 애플리케이션 서버의 스레드 수가 증가
- 이와같은 서비스 불가 상황이 발생하지 않도록 하려면 서버에서 생성되는 스레드 수를 제한하는 방법인 스레드 풀링을 사용해야함
- 이와 같은 구조에서는 동접 사용자 수가 스레드 풀에 지정된 수에 의존하게 되는데, 동접을 최대 허용하기 위해 두 가지 관점에서 생각해 볼 필요가 있다.
    - 자바의 가비지 컬렉션
    - 컨텍스트 스위칭


```java
private void startEchoServer() {
   try (
      Selector selector = Selector.open();
      ServerSocketChannel serverSocketChannel = ServerSocketChannel.open()
    ) {

      if ((serverSocketChannel.isOpen()) && (selector.isOpen())) {
         serverSocketChannel.configureBlocking(false);
         serverSocketChannel.bind(new InetSocketAddress(8888));

         serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
         System.out.println("접속 대기중");

         while (true) {
            selector.select();
            Iterator<SelectionKey> keys = selector.selectedKeys().iterator();

            while (keys.hasNext()) {
               SelectionKey key = (SelectionKey) keys.next();
               keys.remove();

               if (!key.isValid()) {
                  continue;
               }

               if (key.isAcceptable()) {
                  this.acceptOP(key, selector);
               }
               else if (key.isReadable()) {
                  this.readOP(key);
               }
               else if (key.isWritable()) {
                  this.writeOP(key);
               }
            }
         }
      }
      else {
         System.out.println("서버 소캣을 생성하지 못했습니다.");
      }
   }
   catch (IOException ex) {
      System.err.println(ex);
   }
}
```

- 논블로킹 소켓은 Selector 스레드 하나가 모든 클라이언트에 대응한다.
- 서버/클라이언트 동작을 이벤트로 변환

#### 이벤트 루프

- 이벤트 루프란 이벤트를 실행하기 위한 무한루프 스레드
- 객체에서 발생한 이벤트는 이벤트 큐에 입력되고 이벤트 루프는 이벤트 큐에 입력된 이벤트가 있을 때 해당 이벤트를 꺼내서 이벤트를 실행
- 이벤트 루프가 지원하는 스레드 종류에 따라서 단일 스레드 이벤트 루프와 다중 스레드 이벤트 루프로 나뉨

[![이벤트루프](https://github.com/leeplay/study/blob/master/image/%EC%9D%B4%EB%B2%A4%ED%8A%B8%EB%A3%A8%ED%94%84.png?raw=true)]

#### 네티 이벤트 루프

- 네티의 이벤트는 채널에서 발생한다.
- 이벤트 루프 객체는 이벤트 큐를 가지고 있다.
- 네티의 채널은 하나의 이벤트 루프에 등록된다.
- 네티는 리액터/퓨처 패턴 지원


```JAVA
SingleThreadEventExecutor

public abstract class SingleThreadEventExecutor extends AbstractEventExecutor {
    private static final Runnable WAKEUP_TASK = new Runnable() {
        @Override
        public void run() {
            // Do nothing.
        }
    };

    private final EventExecutorGroup parent;
    private final Queue<Runnable> taskQueue;
    final Queue<ScheduledFutureTask<?>> delayedTaskQueue = new PriorityQueue<ScheduledFutureTask<?>>();

	protected Queue<Runnable> newTaskQueue() {
        return new LinkedBlockingQueue<Runnable>();
    }

    protected Runnable pollTask() {
        assert inEventLoop();
        for (;;) {
            Runnable task = taskQueue.poll();
            if (task == WAKEUP_TASK) {
                continue;
            }
            return task;
        }
    }

    protected boolean runAllTasks() {
        fetchFromDelayedQueue();
        Runnable task = pollTask();
        if (task == null) {
            return false;
        }

        for (;;) {
            try {
                task.run();
            } catch (Throwable t) {
                logger.warn("A task raised an exception.", t);
            }

            task = pollTask();
            if (task == null) {
                lastExecutionTime = ScheduledFutureTask.nanoTime();
                return true;
            }
        }
    }
 ```


#### 채널 파이프라인

- 채널파이프라인은 채널에서 발생한 이벤트가 이동하는 통로
- 이 통로를 통해서 이동하는 이벤트를 처리하는 클래스를 이벤트 핸들러라고하면 이벤트 핸들러를 상속받아서 구현한 구현체들을 코덱이라함

[![pipeline](https://github.com/leeplay/study/blob/master/image/pipeline.png?raw=true)]()

	public static void main(String[] args) throws Exception {
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap b = new ServerBootstrap();
            b.group(bossGroup, workerGroup)
             .channel(NioServerSocketChannel.class)
             .childHandler(new ChannelInitializer<SocketChannel>() {
                @Override
                public void initChannel(SocketChannel ch) {
                    ChannelPipeline p = ch.pipeline();
                    p.addLast(new EchoServerV1Handler());
                }
            });

            ChannelFuture f = b.bind(8888).sync();
            f.channel().closeFuture().sync();
        }
        finally {
            workerGroup.shutdownGracefully();
            bossGroup.shutdownGracefully();
        }
    }


[![inoutboundhandler](https://github.com/leeplay/study/blob/master/image/inoutboundhandler.png?raw=true)]()


#### Reactor 개념 (네티는 이 모델의 구현체 입니다.)

[![vertx1](https://github.com/leeplay/study/blob/master/image/vertx1.png?raw=true)]()


- 이벤트에 반응하는 reactor를 만들고 reactor에 이벤트를 처리할 event handler들을 등록한다. (initiate)
- reactor는 이벤트가 발생하기를 기다린다. (receive)
- 이벤트가 발생하면 이벤트를 처리할 event handler단위로 분할한다.(demultiplex)
- 분할된 이벤트를 해당 event handler에게 발송한다. (dispatch)
- event handler에 알맞은 method를 사용하여 이벤트를 처리한다. (process event)


#### Reactor 개요

- 이벤트 기반 비동기 어플리케이션을 개발할 때 반드시 지켜야 할 규칙은 Initiation Dispatcher가 실행되는 스레드를 블록시키거나 시간이 오래 걸리는 작업을 실행하느라 이벤트 처리가 지연되지 않게끔 해야함 (골든룰)
- Reactor 동시 처리 모델은 연결되는 TCP 채널의 개수와 관계없이 항상 일정한 스레드 개수가 유지되어 불필요한 오버헤드가 없음
- 일반적인 Reactor 동시 처리 모델은 단일 스레드에서 동작하여 병렬 처리의 이점을 얻을 수 없다는 단점이 있지만 MultiReactor 패턴으로 이러한 단점을 극복했다. 

#### 내부모델 1

<img width="618" alt="2019-02-26 6 00 39" src="https://media.oss.navercorp.com/user/375/files/e3c0e6a0-39f0-11e9-8a63-f08077f846e6">

#### 내부모델 2

<img width="890" alt="2019-02-26 6 00 58" src="https://media.oss.navercorp.com/user/375/files/e6dfbd48-39f0-11e9-92bc-8a197495cb74">


### 쓸데없는 용어 정리

- 이 용어들은 io모델의 변천사에 영향받은 듯

#### 논블로킹 웹프레임워크의 쓰레드 구분

- selector = acceptor = boss thread
  - 용어는 다양한데 소켓 프로그래맹의 accept를 수행하는 쓰레드 입니다.
- worker = eventloop = eventloop worker thread ?
- 별도 블로킹 잡 처리 쓰레드풀은 직접 명명함 (버텍스에서는 헷갈리게 애를 worker라고 부름)

#### 웹플룩스 내 쓰레드 명명

- reactor-http-nio =  reactor-http-server-epoll
  - http request를 받은 후, 요청/서비스 로직까지 처리하는 working thread
  - I/O Multiplexing 구현하기 위한 시스템 호출로 select, poll, epoll, kqueue가 있음
  - os, kernel 차이로 네티에서 어떤 옵션을 사용할지 선택함
- reactor-http-client-epoll : WebClient 에서 외부 IO 를 처리하는 thread
- 명명의 의미를 생각해보면 구조의 신뢰를 주는 명명임

#### 쓰레드 수 결정

- 기본적으로 개발자가 설정 가능함
- 하지만 잘하지 않음, 왜 안해도 될까? 성능검사 참조

<img width="815" alt="2019-02-27 3 30 54" src="https://media.oss.navercorp.com/user/375/files/c8c87d8a-3aa5-11e9-86a2-983558205760">


- 설정하지 않았다면 보통 아래 룰로 설정됨

```
int DEFAULT_IO_WORKER_COUNT = Integer.parseInt(System.getProperty(
			"reactor.ipc.netty.workerCount",
			"" + Math.max(Runtime.getRuntime()
			            .availableProcessors(), 4)));
```

```
int DEFAULT_IO_SELECT_COUNT = Integer.parseInt(System.getProperty(
			"reactor.ipc.netty.selectCount",
			"" + -1));
```

- 보통 상황에선 쓰레드들의 역할을 알 필요 없음
- 성능테스트나 병목 발생 시 상황을 해석할 경우 쓰레드들의 역할을 알아야 함
- 왜냐하면 기존에 우리가 생각하는 톰캣 방식처럼 구동하지 않음


#### 성능

#### servlet vs netty

- 백엔드 없이 동작

|  | servlet | netty |
| --- | --- | --- |
| v3000-tps | 평균 : 18,674.6 | 평균 : 20,078.3|
| 성공/실패카운트 | 3,295,827/1,136 | 3,502,522/37 |
- cpu
  - servlet -> netty 순 그래프 입니다.

<img width="850" alt="2017-03-09 12 04 49" src="https://media.oss.navercorp.com/user/375/files/b4cff6c2-04c0-11e7-937e-06ad12a1dbab">

- 백엔드 즉시 리턴

|  | servlet | netty |
| --- | --- | --- |
| v3000-tps | 평균 : 6,700.4 | 평균 : 11,416 |
| 성공/실패카운트 | 1,169,009/103,146 | 1,992,031/0 |


#### netty cpu core

|count|tps|
| --- | --- |
| eventloop-1 |9,099 tps |
| eventloop-8 |19,485 tps |
| eventloop-16 |19,460 tps |
| eventloop-32 |19,754 tps |
| eventloop-64 |19,914 tps |
| eventloop-128 |19,183 tps |

- 이벤트루프 프로그래밍이 극단적으로 톰캣과 설정이 달라도 되는지 확인할 수 있음
- 왜 웹플룩스가 디폴트를 권고하는지 알 수 있음

#### netty vs netty container

- 이전테스트에서 최적값을 도커라이징함
- 대략 성능 20% 감소

|  | servlet/netty |
| --- | --- |
| v3000-tps |평균 : 16,813|
| 성공/실패카운트 | 2,967,338/0 |


- 다커는 호스트의 호스트의 모든 컨테이너를 브릿지하고, NAT를 통해 외부 네트워크와 연결해 현저하게 전송 단계를 증가시킴
- vhost-net은 전송에서는 아주 효율적이지만 수신 쪽에서는 높은 오버헤드를 가짐
- NAT를 사용하지 않는 컨테이너는 native linux와 동일한 성능을 가지게 될 것이라고 예상 -> 그래서 macvlan을 씁니다. path없이 바로 컨테이너에 연결됨

#### webflux 공식문서

- 위의 내용의 반복입니다. 위를 이해했다면 당연한 말을 합니다.

The key expected benefit of reactive and non-blocking is the ability to scale with a small, fixed number of threads and less memory. That makes applications more resilient under load, because they scale in a more predictable way. In order to observe those benefits

- 고정 쓰레드 사용으로 적은 메모리로 확장 가능
- 부하에 효율적 대응


Performance has many characteristics and meanings. Reactive and non-blocking generally do not make applications run faster. for example, if using the WebClient to execute remote calls in parallel)

- 리액티브 및 논블로킹이 일반적으로 애플리케이션을 더 빠르게 실행하지는 않음

In Spring WebFlux (and non-blocking servers in general), it is assumed that applications do not block, and, therefore, non-blocking servers use a small, fixed-size thread pool (event loop workers) to handle requests.

- 적은 수의 고정 쓰레드 (이벤트 루프 워커스라고 명명, 네티 용어 합쳐서 사용한 듯)

"To scale” and “small number of threads "may sound contradictory but to never block the current thread (and rely on callbacks instead) means that you do not need extra threads, as there are no blocking calls to absorb

- 추가 쓰레드가 필요없다. 


On a “vanilla” Spring WebFlux server (for example, no data access nor other optional dependencies), you can expect one thread for the server and several others for request processing (typically as many as the number of CPU cores). Servlet containers, however, may start with more threads (for example, 10 on Tomcat), in support of both servlet (blocking) I/O and servlet 3.1 (non-blocking) I/O usage.

- cpu core  수에 이벤트 워커 수를 사용하며 톰캣보다 적합한 분야가 있다.
- 톰캣을 이기겠다가 아니라 분야가 다른 것임

Data access libraries and other third party dependencies can also create and use threads of their own. 

- 데이터 처리는 우리 소관이 아니다. 타사 라이브러리를 믿어라
- 이 말은 본인이 외부 io 라이브러리를 사용할 때 스스로 판단가능한 레벨이어야 함, 사용하려는 라이브러리가 논블로킹을 흉내내는 건지 진짜 논블로킹인지
- 사실 되게 무서운 말입니다. 
  - 이벤트를 처리하기 쓰레드들이 소량으로 별도로 생성되지 않고 풀링으로 수백개가 생성됐다면 ???
  - 그래서 webclient를 사용하면 소량으로 이벤트 루프 기반으로 쓰레드가 생성된 겁니다. (그냥 생긴게 아님, 날 믿어라, 하지만 여기까지다)
  
  #### 논블로킹의 오해 (별책부록)

- 톰캣보다 빠르다 ?
  -> 큰 효과가 없을 수도 있다. 나만 논블로킹이면 의미가 없다.
  -> 하지만 아주 견고하다. (큐방식이여서 요청을 놓치지 않는다.)

- 쓰레드 경합은 피해야 한다 ?
  -> 필요하면 해야한다. (아주 잘...)
  -> 예를 들면 모니터링, 메트릭, 격리, 설정 공유 등 (내가 하지 않아도 라이브러리 단에서 수시로 하고 있다.)

- 컨텍스트 스위칭이 전혀 없다 ?
  -> 컨텍스트 스위칭이 아주아주아주 적게 발생한다.

- 디비 접근에 사용할 수 있다 ?
  -> 논란의 대상

- 기본적으로 단일 쓰레드에서 작동한다고 생각하기 때문에 공통잡을 모든 쓰레드가 수행하고 있다면 성능이 저하된다. 이런 경우는 외부 쓰레드로 분리해줘야 한다. 기본적으로 노멀한 비즈니스만 수행시켜야 한다.
  -> 본인이 수행해야 할 잡의 성격에 대해서 이해도가 높아야 한다. (기술적인 요소가 아닌 업무성격)
  
  <h3>결론</h3>

- 더 나은 웹프레임워크의 출현이 아닌 다른 웹프레임워크의 출현
