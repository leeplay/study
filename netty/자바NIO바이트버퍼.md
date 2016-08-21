### 자바 NIO 바이트 버퍼 

바이트 데이터를 저장하고 읽는 저장소, 배열을 멤버 변수로 가지고 있으며 배열에 대한 읽고 쓰기를 추상화한 메서드를 제공한다.
ByteBuffer, CharBuffer, IntBuffer, ShortBuffer, LongBuffer, FloatBuffer, DoubleBuffer가 있으며 각 바이트 버퍼는 저장되는 데이터형에 따라서 선택하면 된다. 

자바 바이트 버퍼
- 바이트버퍼를 생성하는 메서드는 세가지가 존재한다. 
- allocate : JVM 힙영역에 바이트를 생성한다. 힙범퍼라 불린다.
- allocateDirect : 운영체제의 커널 영역에 바이트 버퍼를 생성한다. 다이렉트 버퍼란 불린다. 힙버퍼에 비해 생성시간은 길지만 뛰어난 성능을 제공한다. 

[![bytebuffer](https://github.com/leeplay/study/blob/master/image/bytefueer.png?raw=true)]()

flip 메서드는 쓰기 작업 완료 이후에 데이터의 처음부터 읽을 수 잇도록 현재 포인터의 위치를 변경하여 읽기에서 쓰기 또는 쓰기에서 읽기로 작업을 전환할 수 있게 된다. 
자바의 바이트 버퍼를 사용할 때는 읽기와 쓰기를 분리하여 생각해야 하며 특히 다중 스레드 환경에서 바이트 버퍼를 공유하지 않아야 한다. 
네티는 이와 같은 자바 바이트 버퍼의 문제점을 해결하기 위해서 읽기를 위한 인덱스와 쓰기를 위한 인덱스를 별도로 제공한다. 

[![flip](https://github.com/leeplay/study/blob/master/image/flip.png?raw=true)]()

#### 네티 바이트 버퍼

자바 바이트 버퍼에 비하여 더 빠른 성능을 제공한다. 
바이트 버퍼 할당과 해제에 대한 부담을 줄여주어 GC에대한 부담을 줄여준다. 

- 별도의 읽기 인덱스와 쓰기 인덱스
- flip메서드 없이 읽기 쓰기 가능
- 가변 바이트 버퍼
- 바이트 버퍼 풀
- 복합 버퍼
- 자바 바이트 버퍼와 네티의 바이트 버퍼 상호 변환 

[![nettybytebuf](https://github.com/leeplay/study/blob/master/image/nettybyte.png?raw=true)]()

네티 바이트 버퍼는 저장되는 데이터형에 따른 별도의 버퍼를 제공하지 않는 대신 각 데이터형에 대한 읽기 쓰기 메서드를 제공한다. 
읽기 작업이 완료된 후에 쓰기 작업을 위해서 flip메서드를 호출할 필요가 없다. 하나의 버퍼에 대하여 쓰기/읽기 작업 병행이 가능하다.

#### 네티 바이트 버퍼 생성

네티 바이트 버퍼는 자바 바이트 버퍼와 달리 프레임워크 레벨의 바이트 버퍼 풀을 제공한다. 
이를 통해 생성된 바이트 버퍼를 재사용한다. 네티의 바이트 버퍼를 바이트 버퍼 풀에 할당하려면 ByteBufAllocator 인터페이스를 사용하면 된다. 
즉 하위 추상 구현체인 PooledByteBufAllocator 클래스로 각 바이트 버퍼를 생성한다. 

- PooledHeapByteBuf, UnpooledHeapByteBuf
- PooledDirectByteBug, UnpooledDirectByteBug
- ByteBufAllocator.DEFAULT.heapBuffer(), Unpooled.buffer()
- ByteBufAllocator.DEFAULT.directBuffer(), Unpooled.directBuffer()

#### 네티 바이트 버퍼 사용

자바 바이트 버퍼는 버퍼를 생성할 때 크기를 지정해야 하며 한 번 생성된 바이트 버퍼의 크기를 변경할 수 없다. 
그에 반해서 네티 바이트 버퍼는 생성된 버퍼의 크기를 동적으로 변경할 수 있다. 

자바의 바이트 버퍼는 언어자체에서 제공하는 것이 없다. 네티는 프레임워크 버퍼 풀을 제공하고 있으며 다이렉트 버퍼와 힙 버퍼를 모두 풀링할 수 있다. 
풀링을 사용하면 GC에 큰 도움이 된다. 네티는 바이트 버퍼를 풀링하기 위해서 바이트 버퍼에 참조 수를 기록한다. 네티는 바이트 버퍼의 참조 수를 관리하기 위하여 
ReferenceCountUtil 클래스에 정의된 retain, release메서드를 사용한다.  

네티 바이트 버퍼는 nioBuffer 메서드를 사용하여 자바 NIO버퍼로 변환이 가능하다. 
