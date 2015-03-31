
Networkd configurations 
=======================

[![Network configurations](https://github.com/leeplay/study/blob/master/etc/nicstack.PNG?raw=true)]()

- 결론 : Native에 비해 성능 저하의 주요인은 NAT 사용으로 인한 path 의 증가 때문이다.  


Network bandwidth
==================

[![Network bandwidth](https://github.com/leeplay/study/blob/master/etc/bandwidth.PNG?raw=true)]()

- 1500 byte MTU 단일 TCP 연결 위로 단방향의 벌크 데이터 전송의 goodput을 측정하기 위해 nuttcp를 사용
- 10 gbps 로 튜닝함(socket buffer size 증가, tcp window scaling)
- 다커는 호스트의 호스트의 모든 컨테이너를 브릿지하고, NAT를 통해 외부 네트워크와 연결해 현저하게 전송 단계를 증가시킴
- vhost-net은 전송에서는 아주 효율적이지만 수신 쪽에서는 높은 오버헤드를 가짐(kernel-user space의 context switching을 회피하기 때문에 network bandwidth 성능이 개선되었다고 알려져있음)
- NAT를 사용하지 않는 컨테이너는 native linux와 동일한 성능을 가지게 될 것이라고 예상 

Network latency
================

[![Network latency](https://github.com/leeplay/study/blob/master/etc/latency.PNG?raw=true)]()

- 네트워크 지연시간 측정을 위해 netperf를 사용  
- NAT를 사용하는 docker는 native에 비해 2배의 지연속도를 가짐
- KVM은 가상화 되지않은 네트워크 스택과 비교해 80%증가한 30ms 의 부하를 가짐
- 두 경우 모두 트랜잭션이 각 방향으로 하나의 패킷으로 구성하기 때문에 TCP 및 UDP는 매우 유사한 지연시간을 가짐


History of Docker network performance
==========================

- deview 2013에 소개된 docker 0.6 benchmark 

[![docker 2013](https://github.com/leeplay/study/blob/master/etc/docker-benchmark-2013.PNG?raw=true)]()

- 위의 테스트 결과(0.6 ~ 1.0)를 종합했을 때 전혀 속도 개선이 이뤄지지 않았다.


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

- 1000번의 NAT가 수행된다면 대략 17 퍼센트 정도의 성능 저하가 발생합니다.
- n초당 NAT 횟수가 증가하면 라우터의 forward 능력이감소됩니다.  
- translation table의 증가는 라우터가 만들 수 있는 새로운 translation entry가 감소합니다.
- translation entry를 수행하기 위해 필요한 CPU 점유율 증가합니다. 

[![NATCPU](https://github.com/leeplay/study/blob/master/etc/nat-cpu-utilization.png?raw=true)]()

참고문헌 
=======

- IBM Research Report - An Updated Preformance Comparison of Virtual Machines and Linux Containers (http://domino.research.ibm.com/library/cyberdig.nsf/papers/0929052195DD819C85257D2300681E7B/$File/rc25482.pdf)

- Impact of Network Address Translation on Router Performance (https://vtechworks.lib.vt.edu/handle/10919/35307)
