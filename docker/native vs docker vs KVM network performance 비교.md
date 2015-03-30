
Networkd configurations 
=======================

[![Network configurations](https://github.com/leeplay/study/blob/master/etc/nicstack.PNG?raw=true)]()

- Native에 비해 성능 저하의 주요인은 path 의 증가 때문이다.  
- 아래의 테스트는 docker 1.0 에서 테스트되었습니다.

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


History of Docker network performance
==========================

- deview 2013에 소개된 docker 0.6 benchmark 

[![docker 2013](https://github.com/leeplay/study/blob/master/etc/docker-benchmark-2013.PNG?raw=true)]()

- 위의 테스트 결과를 종합했을 때 전혀 속도 개선이 이뤄지지 않았다.


### docker network 성능저하 이유

- qdisc (kernel bug)
- RPS/RFS 
- linuxbridge
- NAT


### qdisc 

- 패킷을 생성해서 드라이버로 패킷을 내려 보낸다. 드라이버의 앞 부분에는 큐(queue)가 있다. 패킷은 우선 큐에 들어가고, 큐 구현체가 패킷이 드라이버로 전달되는 시점을 결정한다. Linux의 qdisc(queue discipline)가 이것이다.
- Linux traffic control 기능은 qdisc를 조작하는 것이다. 기본으로 사용하는 qdisc는 단순한 FIFO(first-in-first-out) 큐이다.
- linux 진영에서 veth의 qdisc의 size를 0으로 설정해야 한다는 patch가 올라옵니다. 하지만 master에는 merge 안됨 (http://patchwork.ozlabs.org/patch/396190/)
- tencent 개발자가 veth의 qdisc의 size를 0으로 설정해 qdisk를 사용하지 않게하는 병목현상 개선 pr을 날립니다.  (https://github.com/docker/libcontainer/pull/193)
- pr은 libcontainer-1.4.0 에서 merge 됩니다. (https://github.com/docker/libcontainer/pull/221/files)
- libcontainer-1.4.0은 2014년 12월 15일 이후에 릴리즈되었기 때문에 docker 최신버전에서 network performance의 개선이 있을 것이라 예상됩니다. 하지만 최신버전에 대한 성능 테스트의 자료가 없어 직접 수행해봐야 할 거 같습니다.


### RPS/RFS

[![NORPS](https://github.com/leeplay/study/blob/master/etc/NoRPS.PNG?raw=true)]()
[![RPS/RFS](https://github.com/leeplay/study/blob/master/etc/rfs.PNG?raw=true)]()

- 중국인 개발자에 의해 최초 의문이 제기됨 (https://github.com/docker/docker/issues/8277)
- 성능 개선을 위해 veth에 RPS와 RFS 설정을 하였는데도 송수신 성능저하가 발생한다는 의견
- docker 측에서는 이 이슈를 qdisc 이슈로 처리함


### linuxbridge

- linuxbridge는 ovs의 비해 느리다. (http://www.opencloudblog.com/?p=96)
- docker 커뮤니티에서는 container가 ovs를 바로 사용하는 것에 대해 활발히 논의 중이다. (https://github.com/docker/docker/issues/8951), (https://github.com/docker/docker/issues/9983)


### NAT

```
With NAT enabled, there is a throughput performance degradation of approximately 17% with
1,000 NAT entries for an input stream containing 64 byte packets. The baseline throughput was
determined to be 36,625 bytes/s when NAT was not configured on the system. With 1,000 NAT
entries the throughput degraded to 30,250 bytes/s, which represents a performance degradation of
approximately 17%. As the number of translation entries increases, the number of packets per
second the router can forward decreases.
The rate at which the router can create new translation entries decreases as the number of NAT
translation entries in the translation table increases. A corollary to this characteristic is, with a
fixed packet rate (throughput), as the number of translation entries increases the percentage of
CPU required to produce new entries increases. Four sets of tests were used to test this
characteristic. One aspect of these tests determined how fast a router could create a specified
number of new NAT entries without dropping a packet. The second aspect of these set of tests
was to determine the impact on CPU utilization with an increase in the number of NAT entries.
35
CPU utilization was monitored while the number of NAT entries increased. The CPU utilization
increased with the number of entries as in Table 4.4. The rate at which 100% CPU utilization is
reached depends on the rate of creation of new translation entries and how many translation
entries are already in the NAT translation table. For example, the packet rate was limited to 150
packets per second (PPS) and the CPU utilization climbed as the number of entries increased in
the NAT translation table. At 100% utilization, only 40 new translations-per-second could be
created when the NAT table contained 10,000 entries.
As the number of translation entries in the translation table increases, the amount of memory used
increases. The maximum number of NAT entries depends only on the amount of memory in the
router. As the number of embedded applications supported by NAT increases in more recent
versions of router operating system images, the number of bytes per entry will increase due to
flags and other required data. In the router’s Operating System (OS) image that was used in the
tests for this thesis, memory utilization was determined to be 124 bytes per NAT entry.
IOS NAT is a performance-intensive feature that performs read and write operations on packets.
As a result of this, moderate to heavy performance degradation is observed in networks. Since the
NAT is a single point of entry and exit into a network, the NAT box happens to be a performance
bottleneck. The next section addresses the enhancements that can be considered to the NAT
software to optimize system performance.
```




Impact of NAT on Router Performance  
===================================


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
