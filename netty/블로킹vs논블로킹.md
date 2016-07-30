
### 동기/비동기
동기 : 서비스 처리가 완료된 이후에 처리 결과를 알 수 있는 방식을 동기식 호출 
비동기 : 요청에 대한 결과를 수신하지 못 했음에도 요청이 완료되었기 때문에 결과를 기다리는 시간에 다른 작업을 수행할 수 있고 필요한 시기에 처리 결과를 확인할 수 있다.
       비동기를 지원하는 디자인 패턴은 퓨처, 프로미스, 옵저버 패턴, Node.js는 콜백 함수, 네티는 리액터 패턴을 사용

### 블로킹/논블로킹
소켓의 동작 방식은 블로킹, 논블로킹 모드로 나뉜다. 
블로킹은 요청한 작업이 성공하거나 에러가 발생하기 전까지는 응답을 돌려주지 않으며 논블로킹은 요청한 작업의 성공 여부와 상관없이 바로 결과를 돌려주는 것을 말한다.
이때 요청의 응답값에 의해서 에러나 성공여부를 판단한다. 

블로킹 소켓은 ServerSocket, Socket클래스
논블로킹 소켓은 ServetSocketChannel, SocketChannel 클래스를 사용한다.

#### 블로킹

블로킹 소켓은 데이터 입출력에서 스레드의 블로킹이 발생하기 때문에 동시에 여러 클라이언트에 대한 처리가 불가능하게 된다. 
클라이언트가 서버에 접속하면 서버 소켓의 accept메서드를 통해서 연결된 클라이언트 소켓을 얻어온다. 
이때 블로킹 소켓은 I/O 처리에 블로킹이 발생하기 때문에 새로운 스레드를 하나 생성하고 그 스레드에게 클라이언트 소켓에 대한 I/O 처리를 넘겨주게 된다. 
이로써 서버소켓이 동작하는 스레드는 다음 클라이언트의 연결을 처리할 수 있게 된다. 

blocking.jpg

서버 소켓의 accept 메서드가 병목 지점이다. 

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

서버 소켓의 accept가 병목지점이다. 블로킹 모드이기 때문에 여러 클라이언트가 동시에 접속 요청을 하는 상황에서 대기시간이 길어진다는 단점이 있다. 
또한 접속할 클라이언트 수가 증가하면 애플리케이션 서버의 스레드 수가 증가하게 되는데 이때 자바의 힙 메모리 부족으로 OOM이 발생할 수 있다. 
이와같은 서비스 불가 상황이 발생하지 않도록 하려면 서버에서 생성되는 스레드 수를 제한하는 방법인 스레드 풀링을 사용하기도 한다. 
클라이언트가 서버에 접속하면 서버 소켓으로부터 클라이언트 소켓을 얻어온다. 
다음으로 스레드 풀에서 가용 스레드를 하나 가져오고 해당 스레드에 클라이언트 소켓을 할당한다. 
이후부터는 클라이언트 소켓에서 발생하는 I/O 처리를 할당된 스레드가 전담하게 된다. 이와 같은 구조에서는 동접 사용자 수가 스레드 풀에 지정된 수에 의존하게 된다. 
동접을 최대 허용하기 위해 두 가지 관점에서 생각해 볼 필요가 있다. 

첫 번째는 자바의 가비지 컬렉션에 대한 고찰이다. 힙 크기가 크면 클수록 가비지 컬렉션에 드는 시간이 길어진다는 점이다. 즉 힙 크기와 생성 가능한 스레드 수는 비례한다. 
힙에 할당된 메모리가 크면 클수록 ㅏㄱ비지 컬렉션이 수행되는 횟수는 줄어들지만 수행시간은 상대적으로 길어진다. 

두 번째는 운영체제에서 사용되는 컨텍스트 스위칭에 대한 고찰이다. 수 많은 스레드가 CPU자원을 획득하기 위해 경쟁하면서 CPU자원을 소모하기 때문에 실제로 작업에 사용할 CPU자원이 적어지게 된다. 

#### 논블로킹
앞에서 살펴본 블로킹 모드의 소켓은 입출력 메서드가 호출되면 처리가 완료될 때까지 다른 처리를 할 수 없었다. 이런 단점을 해결하는 방식이 논블로킹 소켓이다. 

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

논블로킹 소켓은 구조적으로 소켓으로부터 읽은 데이터를 바로 소켓에 쓸 수가 없다. 이를 위해서 각 이벤트가 공유하는 객체를 생성하고 그 객체를 통해서 각 소켓 채널로 데이터를 전송한다.
블로킹 소켓은 각 소켓 스레드가 각 클라이언트에 대응을 하고, 논블로킹 소켓은 Selector 스레드 하나가 모든 클라이언트에 대응한다. 

nonblocking.jpg

#### 이벤트 기반 프로그래밍
각 이벤트를 먼저 정의해두고 발생한 이벤트에 따라서 코드가 실행되도록 프로그램을 작성하는 것, 논블로킹도 이벤트 기반 프로그램의 한 종류다.
서버/클라이언트 동작을 이벤트로 변환하면 이벤트 기반 네트워크 프로그래밍을 제공할 수 있다. 소켓을 직접 다루게 되면 소켓에 문제가 발생하거나 연결된 스트림에 문제가 발생 시 
데이터를전송/소켓생성 하는 예외처리 코드가 작동되게 해야 한다. 이와같은 구조는 코드의 중복을 야기시키고 디버깅에 혼란을 초래할 수 있다.
네티에서는 데이터 읽기/쓰기를 위한 이벤트 핸들러를 지원한다. 데이터를 소켓에 직접 기록하는 것이 아닌 데이터 핸들러를 통해 기록해 서버의 코드를 클라이언트에서 재사용하는 장점과 
이벤트와 로직을 분리하는 장점을 제공한다. 에러 이벤트도 정의되어 에러 처리에 대한 부담을 줄여준다. 

