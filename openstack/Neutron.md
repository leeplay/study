뉴트론을 들어가기 전 뉴트런에서 사용하는 인스턴스 네트워크 가상화, 네트워크 타입, 라우팅, 보안정책에 대한 선수지식이 필요합니다. 


Network Namespace
=================

- 네트워크 스택에 대한 chroot 와 같은 역할
- 네트워크 장치, 주소, 포트, 라우트, 방화벽 규칙 등의 사용을 분할하여 인스턴스가 실행 중이 환경에서 네트워크를 가상화함
- 네트워크 가상화의 가장 기본 기술 (OpenStack, Docker에서 사용함)
- 커널 2.6.24 버전에서 추가 

### 실습

- namespace 생성 (/var/lib/netns 경로에 생성됩니다.)
- neutron (https://github.com/openstack/neutron/blob/592f641b4673b07737d689187bc999dff9e4a502/neutron/agent/linux/ip_lib.py#L130)

[![openstack](https://github.com/leeplay/study/blob/master/etc/2015-03-05%2018;46;36.PNG?raw=true)]()


- docker (https://github.com/docker/docker/blob/00d19150bb937bcc4572edf1f397d4051abb37c1/docs/sources/articles/runmetrics.md)

```
# ip netns add nexhubns
```

- namespace 내에서 보이는 인터페이스 조회 
- 최초에는 루프백 장치만 있음 
- 각 네트워크 장치는 오직 한 개의 단일 네트워크 네임스페이스에만 소속됨 
- 실제 물리 장치들은 root 네임스페이스가 아닌 어떤 네임스페이스에도 할당될 수 없다. 
- 가상 네트워크 장치들은 (eg. veth) 네임스페이스에 할당될 수 있다.
- 이 가상 장치들이 프로세스들이 네임스페이스 내에서만 네트워크 통신을 수행하도록 지원한다.

```
# ip netns exec nexhubns ip link list

root@kyu-HP-EliteBook-2570p:/run/netns# ip netns exec nexhubns ip link list
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```

- nexhubns와 root 네임스페이스와는 통신이 불가능한 상태 -> 가상 ethernet 장치를 생성해야 함 (eg. docker0)
- 가상 ethernet(eg. nexhub0) 장치 한 쌍을 컨테이너의 가상 ethernet(eg. veth1) 과 맵핑함
- 맵핑된 컨테이너의 가상 이더넷 카드를 네트워크 네임스페이스(eg. nethubns)에 설정함
- 가상 네트워크 장치 시작  


```
# ip link add nexhub0 type veth peer name veth1
# ip link set veth1 netns nexhubns
# ip netns exec nexhubns ifconfig veth1 10.1.1.1/24 up
# ifconfig nexhub0 10.1.1.2/24 up 
# ip netns exec nexhubns ip link list

1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
14: veth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 42:df:cf:bc:f8:44 brd ff:ff:ff:ff:ff:ff
    
# ifconfig 

nexhub0   Link encap:Ethernet  HWaddr 56:5d:e1:9a:09:80
          inet addr:10.1.1.2  Bcast:10.1.1.255  Mask:255.255.255.0
          inet6 addr: fe80::545d:e1ff:fe9a:980/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:8 errors:0 dropped:0 overruns:0 frame:0
          TX packets:132 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:648 (648.0 B)  TX bytes:26023 (26.0 KB)
```


- 통신 확인
- nexhub0과 nexhubns안에 생성된 veth1과 연결된 걸 확인
- routing 정보 확인 (nexhub0은 라우팅 설정은 안된 상태여서 외부와 통신불가)

```
# ping 10.1.1.1
PING 10.1.1.1 (10.1.1.1) 56(84) bytes of data.
64 bytes from 10.1.1.1: icmp_seq=1 ttl=64 time=0.072 ms
64 bytes from 10.1.1.1: icmp_seq=2 ttl=64 time=0.044 ms
64 bytes from 10.1.1.1: icmp_seq=3 ttl=64 time=0.060 ms
64 bytes from 10.1.1.1: icmp_seq=4 ttl=64 time=0.046 ms
64 bytes from 10.1.1.1: icmp_seq=5 ttl=64 time=0.062 ms
64 bytes from 10.1.1.1: icmp_seq=6 ttl=64 time=0.047 ms

# ip netns exec nexhubns ping 10.1.1.2

PING 10.1.1.2 (10.1.1.2) 56(84) bytes of data.
64 bytes from 10.1.1.2: icmp_seq=1 ttl=64 time=0.047 ms
64 bytes from 10.1.1.2: icmp_seq=2 ttl=64 time=0.047 ms
64 bytes from 10.1.1.2: icmp_seq=3 ttl=64 time=0.060 ms
64 bytes from 10.1.1.2: icmp_seq=4 ttl=64 time=0.062 ms
```

```
//host newwork namepsace
# route -FC  
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         10.64.51.1      0.0.0.0         UG    0      0        0 eth0
10.64.51.0      *               255.255.255.0   U     1      0        0 eth0
10.1.1.0        *               255.255.255.0   U     0      0        0 nexhub0
172.17.0.0      *               255.255.0.0     U     0      0        0 docker0
192.168.122.0   *               255.255.255.0   U     0      0        0 virbr0

//nexhubns network namespace
# ip netns exec nexhubns route -FC 
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
10.1.1.0        *               255.255.255.0   U     0      0        0 veth1

//host newwork namepsace
# ip route 
default via 10.64.51.1 dev eth0  proto static
10.64.51.0/24 dev eth0  proto kernel  scope link  src 10.64.51.185  metric 1
172.17.0.0/16 dev docker0  proto kernel  scope link  src 172.17.42.1
192.168.122.0/24 dev virbr0  proto kernel  scope link  src 192.168.122.1

//nexhubns network namespace
# ip netns exec nexhubns ip route  
10.1.1.0/24 dev veth1  proto kernel  scope link  src 10.1.1.1

//host newwork namepsace
root@kyu-HP-EliteBook-2570p:/# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
neutron-openvswi-INPUT  all  --  anywhere             anywhere
ACCEPT     udp  --  anywhere             anywhere             udp dpt:domain
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:domain
ACCEPT     udp  --  anywhere             anywhere             udp dpt:bootps
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:bootps

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
neutron-filter-top  all  --  anywhere             anywhere
neutron-openvswi-FORWARD  all  --  anywhere             anywhere
ACCEPT     all  --  anywhere             192.168.122.0/24     ctstate RELATED,ESTABLISHED
ACCEPT     all  --  192.168.122.0/24     anywhere
ACCEPT     all  --  anywhere             anywhere
REJECT     all  --  anywhere             anywhere             reject-with icmp-port-unreachable
REJECT     all  --  anywhere             anywhere             reject-with icmp-port-unreachable
ACCEPT     all  --  anywhere             anywhere             ctstate RELATED,ESTABLISHED
ACCEPT     all  --  anywhere             anywhere
ACCEPT     all  --  anywhere             anywhere

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
neutron-filter-top  all  --  anywhere             anywhere
neutron-openvswi-OUTPUT  all  --  anywhere             anywhere
ACCEPT     udp  --  anywhere             anywhere             udp dpt:bootpc

Chain neutron-filter-top (2 references)
target     prot opt source               destination
neutron-openvswi-local  all  --  anywhere             anywhere

Chain neutron-openvswi-FORWARD (1 references)
target     prot opt source               destination

Chain neutron-openvswi-INPUT (1 references)
target     prot opt source               destination

Chain neutron-openvswi-OUTPUT (1 references)
target     prot opt source               destination

Chain neutron-openvswi-local (1 references)
target     prot opt source               destination

Chain neutron-openvswi-sg-chain (0 references)
target     prot opt source               destination

Chain neutron-openvswi-sg-fallback (0 references)
target     prot opt source               destination
DROP       all  --  anywhere             anywhere             /* Default drop rule for unmatched traffic. */

//nexhubns network namespace
# ip netns exec nexhubns iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination

```



- 삭제 

```
# ifconfig nexhub0 down
# ip netns delete nexhubns
```

### 정리

1. 컨테이너 생성 시 네임스페이스 생성
2. 호스트의 물리 이더넷 장치를 컨테이너 네임 스페이스에서 사용할 수 없기 때문에 네트워크 네임스페이스 안에 호스트용 가상 이더넷 장치와 컨테이너의 이더넷 장치를 연결
3. 호스트용 가상 이더넷 장치를 물리 이더넷 장치로 연결 
4. 이제 아래의 그림이 이제 이해가 되시죠 ~~~ ???

[![Network Namespace](https://github.com/leeplay/study/blob/master/etc/2015-03-06%2011;47;18.PNG?raw=true)]()

###  neutron 에서는 ?

- L2(가상 스위칭), L3(가상 라우팅) 사용을 위해 TUN(NexHubNS)/TAP(Veth1) 디바이스로 사용

[![neutron br-tun](https://github.com/leeplay/study/blob/master/etc/br-tun.PNG?raw=true)]()

VXLAN : 작성 중...
=====

- Cloud가 성장하면서 주목 받게 된 Network-Side 기술
- 네트워크 스위치들의 MAC Addresse Table 수용에 한계 
- 가상화 환경에서 네트워크 단일 도메인의 VLAN 숫자 한계
- 가상화 환경에서 네트워크의 경직된 구성으로 인해 유연한 이동 한계

### 네트워크 스위치들의 MAC Addresse Table 수용에 한계 

- 가상화를 구현하지 않았던 시절에 서버를 L2 단일 스위치 도메인에 수천 대를 연결해도 큰 문제가 있지 않지만, 예를 들어 한 호스트에 30여개의 vm을 생성한다면 60개의 MAC Address 가 생성된다면 수십만 대의 address를 스위치에 연결해야 해 사실상 네트워크 도메인에서 수용불가능한 레벨이 되어버림 
- VXLAN은 불필요하게 모든 MAC Address를 네트워크 스위치에서 가지고 있지 않고, 하단의 가상화 스위치에서만 소유하고 해당 테이블을 통해 Forwarding 하는 방식 

### 가상화 환경에서 네트워크 단일 도메인의 VLAN 숫자 한계
- Ehternet Frame Format이 12 bit의 VLAN ID를 제공 (eg. 4,096개)
- VXLAN은 VLAN ID 가 24 bit 로 제공 (eg. 16,000,000개)
 
### 가상화 환경에서 네트워크의 경직된 구성으로 인해 유연한 이동 한계


IPTables : 작성 중...
========


Neutron : 다음 주 예정 ...
=======

### 설치

- Bad router request: No IPs available for external network 7884eb60-71af-420e-2d1d02f55

[![Bad router request](https://github.com/leeplay/study/blob/master/etc/qrouter.PNG?raw=true)]()

### 스위칭

### 라우팅

### 로드 밸런싱

### 방화벽 

ML2 plugin : 다음 주 예정 ...
==========
