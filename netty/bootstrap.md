### 부트스트랩
네티로 작성한 네트워크 애플리케이션의 동작 방식과 환경을 설정하는 도우미 클래스다.
소켓 옵션 -> 주소/포트 -> 채널 파이프라인 -> 이벤트 루프(단일/다중) -> 소켓모드/IO타입
네티 부트스트랩은 서버를 위한 ServerBootstrap 클래스, 클라이언트를 위한 Bootstrap클래스로 나뉜다. 

[![Bootstrap](https://github.com/leeplay/study/blob/master/image/bootstrap.png?raw=true)]()

각 부트스트랩 클래스는 AbstractBootstrap 클래스와 Channel 인터페이스를 상속받고 있다. 
부트스트랩을 사용하면 ㅇ네트워크 어플리케이션을 작성할 때 유연성을 얻을 수 있다. 
블로킹 소켓을 지원하는 서버를 논블로킹 모드로 전환하는 데 아주 많은 코드를 변경해야 하지만 네티는 
부트스트랩 설정을 통해서 데이터 처리 코드를 변경하지 않고 Blocking, NonBlocking Server 와 동일하게 동작하는 
애플리케이션을 작성할 수 있다. 이와 같이 네티의 부트스트랩은 소켓 채널에 대한 입출력을 우아하게 추상화함으로써 
네티 사용자는 단순히 설정을 변경하기만 하면된다.

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
                   p.addLast(new EchoServerHandler());
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

빌더 패턴을 사용하므로 group, channel과 같은 메서드로 객체를 초기화한다.
NioEventLoopGroup 클래스 생성자의 인수로 사용된 숫자는 스레드 그룹 내에서 생성할 최대 스레드 수를 의미한다. 
인수를 생략하면 사용할 스레드 수를 하드웨어 코어 수를 기준으로 결정한다. 스레드 수는 하드웨어가 가지고 있는 CPU 코어 수의 2배를 사용한다.
NioEventLoopGroup 클래스는 부모 스레드다. 부모 스레드는 연결 요청의 수락을 담당한다. 자식 스레드는 연결된 소켓에 대한 IO처리를 담당한다.

블로킹/EPoll(리눅스만 지원)을 사용하고 싶다면 구현체만 변경하면 된다. 
EpollEventLoopGroup, OioEventLoopGroup

#### group
데이터 송수신을 위한 이벤트 루프를 설정
클라이언트는 연결 요청이 완료된 이후의 데이터 송수신 처리를 위해서 하나의 이벤트 루프만 있으면 됨
서버는 클라이언트의 연결 요청을 수락하기 위한 이벤트 루픝와 데이터 송수신 처리를 위한 이벤트 루프 이렇게 두 종류의 이벤트 루프가 필요하다. 

#### channel 
소켓 입출력 모드 설정

* LocalServerChannel : 하나의 자바 가상머신에서 가상 통신을 위한 서버 소켓 채널을 생성
* OioServerSocketChannel : 블로킹 모드의 서버 소켓 채널을 생성
* NioServerSocketChannel : 논블로킹 모드의 서버 소켓 채널을 생성
* EpollServerSocketChannel : 리눅스 커널의 epoll 입출력 모드를 지원하는 서버 소켓 채널을 생성
* OioSctpServerChannel : SCTP 전송 계층을 사용하는 블로킹 모드의 서버 소켓 채널을 생성
* NioSctpServerChannel : SCTP 전송 계층을 사용하는 논블로킹 모드의 서버 소켓 채널을 생성
* NioUdtByteAcceptorChannel : UDT 프로토콜을 지원하는 논블로킹 모드의 서버 소켓 채널을 생성
* NioUdtMessageAcceptorChannel : UDT 프로토콜을 지원하는 블로킹 모드의 서버 소켓 채널을 생성

SCTP : TCP, UDP와 같은 전송 계층에 해당하는 포로토콜로써 차세대 프로토콜로 주목 받음, 리눅스에서 지원함, 윈도우는 지원안함
UDT : UDP프로토콜을 기반으로 작성된 어플리케이션 계층의 프로토콜이다. 즉 HTTP, SMTP같은 레벨이다. 주로 고성능 분산 컴퓨팅의 데이터 전송에 사용됨 

#### channelFactory 
소켓 입출력 모드 설정
channel메서드와 동일한 기능 수행, ChannelFactory 인터페이스를 상속받은 클래스를 설정

#### handler
서버 소켓 채널의 이벤트 핸들러 설정
서버 소켓 채널에서만 발생하는 이벤트를 수신하여 처리한다. 
.handler(new LoggingHandler(LogLEvel.INFO))

#### childHandler 
소켓 채널의 데이터 가공 핸들러 설정
클라이언트 소켓 채널로 송수신되는 데이터를 가공하는 데이터 핸들러 
이 메서드를 통해서 등록되는 이벤트 핸들러는 서버에 연결된 클라이언트 소켓 채널에서 발생하는 이벤트를 수신하여 처리한다.
 public void initChannel(SocketChannel ch) {
    ChannelPipeline p = ch.pipeline();
    p.addLast(new EchoServerHandler());
    p.addLast(new LoggingHandler(LogLevel.DEBUG));
 }

#### option

서버 소켓 채널의 옵션 지정, 커널에서 사용하는 값을 변경한다는 의미

* TCP_NODELAY : 데이터 송수신에 Nagle 알고리즘의 비활성화 여부를 지정
* SO_KEEPALIVE : 운영체제에서 지정된 시간에 한번씩 keepalive패킷을 상대방에게 전송
* SO_SNDBUF : 상대방으로 송신할 커널 송신 버퍼의 크기 
* SO_RCVBUF : 상대방으로 수신할 커널 수신 버퍼의 크기 
* SO_REUSEADDR : TIME_WAIT 상태의 포트를 서버 소켓에 바인드할 수 있게 한다.
* SO_LINGER : 소켓을 닫을 때 커널의 송신 버퍼에 전송되지 않은 데이터의 전송 대기시간을 지정
* SO_BACKLOG : 동시에 수용 가능한 소켓 연결 요청 수

TCP_NODELAY
네이글 알고리즘은 가능하면 데이터를 나누어보내지 말고 한꺼번에 보내라라는 원칙을 기반으로 만들어진 알고리즘
처음에 작은 크기의 데이터를 전송하면 커널의 송신 버퍼에서 적당한 크기로 모아서 보낸다. ACK을 받아야 다음 패킷을 전송한다.
네트워크 상태가 좋지않고 대역폭이 좁을 때 네트워크를 효율적으로 사용하지만 빠른 응답시간이 필요한 서비스에서는 좋지 않은 결과를 가져온다.

SO_REUSEADDR
자신의 소켓을 종료하겠다는 FIN패킷을 서로 주고받는데 마지막 ACK패킷을 전송한 피어의 소켓 상태가 일정 시간 동안 TIM_WAIT으로 바뀌게 되는데 
이것은 자신이 전송한 ACK패킷이 상대방으로 도달하기를 기다리는 시간이다. 어플리케이션 서버가 강제 종료 또는 비정상 종료로 인해 재시작하는 상황에서
사용하던 포트 상태가 TIME_WAIT에 있다면 어플리케이션 서버는 bind함수가 실패하여 정상 동작하지 못 한다. 

SO_BACKLOG
동시에 수용할 클라이언트의 연결 요청 수를 말하지만 서버 소켓이 수용할 수 있는 동시 연결 수가 아니라는 점
SYN-RECEIVED상태로 변경된 소켓 연결을 가지고 있는 큐의 크기를 설정하는 옵션, 이 값을 크게 설정하면 클라이언트 연결 요청이 폭주할 때 연결 대기 시간이 길어져
클라이언트에서 연결 타임아웃이 발생할 수 있고 너무 작제 잡으면 연결을 못 하는 상황이 발생한다. 

#### childOption
소켓채널의 소켓 옵션 설정
서버에 접소개한 클라이언트 소켓 채너에 대한 옵션을 설정함, 대표적으로 SO_LINGER가 있다.
소켓 close 메서드를 호출한 이후에 제어권은 어플리케이션에서 운영체제로 넘어가게 된다. 이때 커널 버퍼에 아직 전송되지 않으 데이터가 남아 있으면 어떻게 처리할 지 지정하는 옵션 
이 옵션을 켜면 close 메소드가 호출되었을 때 커널 버퍼의 데이터를 상대방으로 모두 전송하고 상대방의 ACK 패킷을 기다린다. 포트 상태가 TIME_WAIT으로 전환되는 것을 방지하기 위해 
SO_LINGER옵션을 활성화하고 타임아웃값을 0으로 설정하는 편법도 있다.
