
Networkd configurations 
=======================

[![Network configurations](https://github.com/leeplay/study/blob/master/etc/nicstack.PNG?raw=true)]()

Network bandwidth
==================

[![Network bandwidth](https://github.com/leeplay/study/blob/master/etc/bandwidth.PNG?raw=true)]()

- 1500 byte MTU 단일 TCP 연결 위로 단방향의 벌크 데이터 전송의 goodput을 측정하기 위해 nuttcp를 사용
- 10 gbps 로 튜닝함(socket buffer size 증가, tcp window scaling)
- 다커는 호스트의 호스트의 모든 컨테이너를 브릿지하고, NAT를 통해 외부 네트워크와 연결해 현저하게 전송 단계를 증가시킴
- vhost-net은 전송에서는 아주 효율적이지만 수신 쪽에서는 높은 오버헤드를 가짐
- NAT를 사용하지 않는 컨테이너는 native linux와 동일한 성능을 가지게 될 것이라고 예상 

Network latency
================

[![Network latency](https://github.com/leeplay/study/blob/master/etc/latency.PNG?raw=true)]()

- 네트워크 지연시간 측정을 위해 netperf를 사용  
- NAT를 사용하는 docker는 native에 비해 2배의 지연속도를 가짐
- KVM은 가상화 되지않은 네트워크 스택과 비교해 80%증가한 30ms 의 부하를 가짐
- 두 경우 모두 트랜잭션이 각 방향으로 하나의 패킷으로 구성하기 때문에 TCP 및 UDP는 매우 유사한 지연시간을 가짐


Action item
===========

- 왜 NAT를 사용하면 더 성능저하가 심해지는가 ?
- KVM에서 사용하는 virtio에 알아보자 
- 왜 KVM은 수신 속도가 NAT보다 더 안좋은가

참고문헌 
=======

[![IBM Research Report - An Updated Preformance Comparison of Virtual Machines and Linux Containers](http://domino.research.ibm.com/library/cyberdig.nsf/papers/0929052195DD819C85257D2300681E7B/$File/rc25482.pdf)]()

원문 
=====

E. Network bandwidth—nuttcp
We used the nuttcp tool to measure network bandwidth between the system under test and an identical machine connected using a direct 10 Gbps Ethernet link between two Mellanox ConnectX-2 EN NICs. We applied standard network tuning for 10 Gbps networking such as enabling TCP window scaling and increasing socket buffer sizes. 

As shown in Figure 1
Docker attaches all containers on the host to a bridge and connects the bridge to the network via NAT. In our KVM configuration we use virtio and vhost to minimize virtualization overhead We used nuttcp to measure the goodput of a unidirectional bulk data transfer over a single TCP connection with standard 1500-byte MTU. In the client-to-server case the system under test (SUT) acts as the transmitter and in the server-to-client case the SUT acts as the receiver; it is necessary to measure both directions since TCP has different code paths for send and receive. All three configurations reach 9.3 Gbps in both the transmit and receive direction, 

very close to the theoretical limit of 9.41 Gbps due to packet headers. Due to segmentation offload, bulk data transfer is very efficient even given the extra layers created by different forms of virtualization. The bottleneck in this test is the NIC, leaving other resources mostly idle. In such an I/O-bound scenario, we determine overhead by measuring the amount of CPU cycles required to transmit and receive data. 

우리는 SUT 와 10gbps로 연결된 mellanox 머신의 네트워크 대역폭을 측정하기 위해 nuttcp 툴을 사용했으며 10 gbps로 네트워크 튜닝을 위해 소켓 버퍼 사이즈 증가와 tcp window scaling을 수행하였습니다. 

다커는 호스트의 호스트의 모든 컨테이너를 브릿지하고, NAT를 통해 외부 네트워크와 연결합니다. 우리는 기본 1500 byte MTU 단일 TCP 연결 위로 단방향의 벌크 데이터 전송의 goodput을 측정하기 위해 nuttcp 를 사용했습니다. c/s 경우에 SUT는 전송자처럼 수행하고 s/c 경우에는 수신자처럼 수행합니다. TCP는 송신과 수신을 위한 다른 코드 패스를 가지고 있기 때문에 위와같은 양방향 측정 방식이 필요합니다. 송수신을 9.3gbps로 끌어올리기 위해 3가지 환경설정이 필요합니다. 

패킷 헤더 때문에 9.41gbps이 이론적 한계에 매우 가깝습니다. 세크멘테이션 오프로드 때문에 다른 가상화 형태로 추가로 생성되는 계층은 벌크 데이터 전송에 꽤 효율적입니다. 이 테스트에서 병목은 NIC 이며 남은 다른 리소스는 유휴 상태 입니다. 우리는 데이터를 송수신하는 데에 필요한 CPU 싸이클의 양을 측정함으로써 오버헤드를 판단해야 합니다. 

Figure 2 shows 
system-wide CPU utilization for this test, measured using perf stat -a Docker’s use of bridging and NAT noticeably increases the transmit path length; vhost-net is fairly efficient at transmitting but has high overhead on the receive side. Containers that do not use NAT have identical performance to native Linux. In real network-intensive workloads, we expect such CPU overhead to reduce overall performance.

Historically, Xen and KVM have struggled to provide line-rate networking due to circuitous I/O paths that sent every packet through userspace. This has led to considerable research on complex network acceleration technologies like polling drivers or hypervisor bypass. Our results show that vhost, which allows the VM to communicate directly with the host kernel, solves the network throughput problem in a straightforward way. With more NICs, we expect this server could drive over 40 Gbps of network traffic without using any exotic techniques.

도커에서 사용하는 브릿징과 NAT는 현저하게 전송 단계를 증가시킵니다. vhost-net은 전송에서는 아주 효율적이지만 수신 쪽에서는 높은 오버헤드를 보여줍니다. NAT를 사용하지 않는 컨테이너는 native linux와 동일한 성능을 가집니다.  실제 집중적인 네트워크 부하량에서 우리는 앞서 언급한 CPU 오버헤드가 전체적인 성능저하를 가져올 것이라고 예상합니다.

역사적으로 Xen과 KVM은 빙돌아서 유저 공간을 통해 모든 패킷을 보내는 I/O 경로 때문에 line-rate를(물리적으로 포트에서 손실없이 전달 가능한 최대 속도) 제공하기 위해 노력했습니다. 이런 요구는 폴링 드라이버나 하이퍼바이저 바이패스 같은 네트워크 가속 기술을 이끌었습니다. 우리의 결과는 vhost가 vm이 kernel 과 직접 연결을 허락해 네트워크 throughput 문제를 간단한 방법으로 해결하는 방법을 보여줍니다. 많은 NIC들을 사용해 우리는 서버에서 특이한 기술을 사용하지 않고 40gpbs 이상의 네트워크 트래픽을 제어할 수 있다고 예상했습니다. 

Network latency—netperf
We used the netperf request-response benchmark to measure round-trip network latency using similar configurations as
the nuttcp tests in the previous section. In this case the system under test was running the netperf server (netserver) and the other machine ran the netperf client.  The client sends a 100-byte request, the server sends a 200-byte response, and the client waits for the response before sending another request. Thus only one transaction is in flight at a time

우리는 네트워크 지연시간 측정을 위해 netperf를 사용했습니다. 클라이언트에서 100byte를 request를 전송하면 서버는 respose로 200byte를 전송했습니다. 그리고 클라이언트는 다른 요청을 보내기 전에 응답을 기다렸습니다. 따라서 하나의 트랜잭션이 한 번의 비행시간이다. 

Figure 3 shows the measured transaction latency for both TCP and UDP variants of the benchmark. NAT, as used in Docker, doubles latency in this test. KVM adds 30 ms of overhead to each transaction compared to the non-virtualized network stack, an increase of 80%. TCP and UDP have very similar latency because in both cases a transaction consists of a single packet in each direction. Unlike as in the throughput test, virtualization overhead cannot be amortized in this case.

Nat를 사용하는 docker는 native에 비해 2배의 지연속도를 보여줍니다. KVM은 가상화 되지않은 네트워크 스택과 비교해 80%증가한 30ms 의 부하를 보여줍니다. 두 경우 모두 트랜잭션이 각 방향으로 하나의 패킷으로 구성하기 때문에 TCP 및 UDP는 매우 유사한 지연시간을 가집니다. 이 테스트의 경우는 throughput 테스트와는 달리 가상화 오버헤드는 상환할 수 없습니다.  
