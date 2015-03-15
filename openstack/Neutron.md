Neutron 을 들어가기 전 뉴트런에서 사용하는 인스턴스 네트워크 가상화, 네트워크 타입, 라우팅, 보안정책에 대한 선수지식이 필요합니다. 


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

VXLAN
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
-  Multicast기반으로 능동적으로 Tree 구성

### VXLAN 이란 ?
- VLAN, Layer2의 더 큰 확정성을 제공
- 기존 Ethernet Frame의 한계를 극복할 수 있는 것은 MAC over IP/UDP header(24bit)가 추가되어 VLAN ID를 새롭게 구성하므로 기존의 VLAN 숫자를 뛰어넘는 구성이 가능 
- VMware, Arista Networks, Cisco 에서 제작
- IETF에서 표준화(RFC 7348)
- VM들이 직접 ARP Table을 보유하고 vSwitch 에서 이런한 Table을 관리
- 학습한 목적지 VM Mac은 Peer to Peer Tunnel로 직접 해당 스위치와 통신 
- 학습하지 못한 목적지 VM MAC은 IP Multicast를 이용하여 목적지 VM MAC주소가 있는 스위치에 전송 
- Encapsulation과 Termination 지점에 이러한 기술을 지원해야 하는 스위치가 필요 (OpenVSwitch)
- Overlay Network 구성이 가능해짐

[![VXLAN header](https://github.com/leeplay/study/blob/master/etc/VXLAN-Headers1.png?raw=true)]()


IPTables
========

리눅스에 내장된 방화벽으로서 시스템 관리자가 네트워크 패킷을 처리하는 방식을 지정하는 룰을 체인으로 구성하여 테이블 형태로 작성

### Table

들어온 패킷에 대해 다음과 같은 테이블에 있는 체인에 나온 룰을 순서대로 따라가면서 패킷에 적용함
 
- Raw : 다른 테이블보다 먼저 적용되는 디폴트 테이블
- Filter : 패키시을 필터링할 때 사용되는 디폴트 테이블
- NAT : 네트워크 주소변환에 사용되는 디폴트 테이블 
- Mangle : 특수한 패킷 변경에 사용되는 디폴트 테이블

### Chain 

체인에 있는 룰은 다른 체인으로 이동시키게 할 수 있으며 필요하다면 이런 동작을 반복할 수 있다. IPTables가 활성화되면 해당 노드로 들어오거나 나가는 모든 패킷이 최소한 하나의 체인에 적용됨 

- PREROUTING : 패킷에 대한 라우팅 결정을 내리기 전에 이 체인이 적용 (Raw, Mangle, NAT 에서 사용)
- INPUT : 패킷이 호스트 머신에 전달될 때 적용 (Mangle, Filter 에서 사용) 
- FOWARD : 라우팅 경로는 결정됐지만 로컬에 전달되지 않는 패킷에 이 체인이 적용 (Mangle, Filter 에서 사용)
- OUTPUT : 호스트 머신에서 보낸 패킷이 이 체인에 적용 (Raw, Mangle, NAT, Filter 에서 사용)
- POSTROUTING : 라우팅 결정이 끝난 패킷이 이 체인에 들어온다. (Mangle, NAT 에서 사용)

### Rule

체인에 있는 룰은 패킷에 대한 매칭 기준을 담고 있다. 경우에 따라 다른 체인이 지정되어 있을 수도 있고 DROP이나 ACCEPT와 같은 판단도 할 수 있다. 패킷이 체인에 들어오면 여기에 속한 모든 룰을 하나씩 탐색한다. 

- ACCEPT : 해당 패킷을 허용하고 어플리케이션으로 전달
- DROP : 별다른 동작 없이 패킷을 무시
- REJECT : 패킷을 무시하면서 패킷을 보낸 쪾으로 에러 메시지를 전달
- LOG : 패킷에 대한 세부 사항을 로깅
- DNAT : 패킷의 목적지 IP를 수정
- SNAT : 패킷의 출발지 IP를 수정
- RETURN : 체인을 호출한 곳으로 리턴

ACCEPT, DROP, REJECT와 같은 결정은 필터 테이블에서 내리며 다음과 같은 매칭 기준이 주로 사용됨 

- -p : TCP, UDP, ICMP 등과 같은 프로토콜에 매칭
- -s : 출발지 IP 주소에 매칭
- -d : 목적지 ip 주소에 매칭
- -sport : 출발지 포트에 매칭
- -dport : 도착지 포트에 매칭
- -i : 패킷이 들어오는 인터페이스에 매칭
- -o : 패킷에 나가는 인터페이스에 매칭

Neutron 스위칭
=======

- 오픈스택 네트워킹의 핵심 기능 중 하나는 인스턴스에서 가상 물리 네트워크 인프라를 동적으로 설정할 수 있게 해주는 것이다. 
- 인스턴스가 처음 부팅될 때 호스트에 대해 tap 인터페이스라는 가상 네트워크 인터페이스를 생성하고 tap  인터페이스를 통해 물리 네트워크에 연결하게 된다. 
- 뉴트론에서는 네트워크 브릿지를 이용해 인스턴스를 연결한다. 두 개 이상의 L2 네트워크를 연결해 하나의 통합(aggregate) 네트워크를 생성한다. 뉴트론에서 브릿지는 물리 인터페이스 한 개와 여러 개의 가상 또는 탭 인터페이스로 구성된다. 브릿지로 동작하려면 물리 네트워크 인터페이스를 무작위(promiscuous) 모드로 전환해야 한다. 

[![linux-bridge](https://github.com/leeplay/study/blob/master/etc/openstack-neutron-linuxbridge-12.png?raw=true)]()

### Devstack Networking Example

```
qbr78e1e93f-27 Link encap:Ethernet  HWaddr da:b0:9a:35:75:45
          inet6 addr: fe80::c8d3:1ff:fe88:6586/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:1111 errors:0 dropped:0 overruns:0 frame:0
          TX packets:108 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:194292 (194.2 KB)  TX bytes:16242 (16.2 KB)
		  
qvb78e1e93f-27 Link encap:Ethernet  HWaddr da:b0:9a:35:75:45
          inet6 addr: fe80::d8b0:9aff:fe35:7545/64 Scope:Link
          UP BROADCAST RUNNING PROMISC MULTICAST  MTU:1500  Metric:1
          RX packets:1342 errors:0 dropped:0 overruns:0 frame:0
          TX packets:838 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:235492 (235.4 KB)  TX bytes:146401 (146.4 KB)
		  
qvo78e1e93f-27 Link encap:Ethernet  HWaddr 6a:fe:84:5e:7c:00
          inet6 addr: fe80::68fe:84ff:fe5e:7c00/64 Scope:Link
          UP BROADCAST RUNNING PROMISC MULTICAST  MTU:1500  Metric:1
          RX packets:838 errors:0 dropped:0 overruns:0 frame:0
          TX packets:1342 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:146401 (146.4 KB)  TX bytes:235492 (235.4 KB)
		  
tap78e1e93f-27 Link encap:Ethernet  HWaddr fe:16:3e:71:3e:de
          inet6 addr: fe80::fc16:3eff:fe71:3ede/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:174 errors:0 dropped:0 overruns:0 frame:0
          TX packets:418 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:500
          RX bytes:20840 (20.8 KB)  TX bytes:50530 (50.5 KB)
		  
qbrada6a901-dc Link encap:Ethernet  HWaddr be:ef:e4:57:96:79
          inet6 addr: fe80::6c36:7dff:fe8b:66c7/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:980 errors:0 dropped:0 overruns:0 frame:0
          TX packets:100 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:172951 (172.9 KB)  TX bytes:14715 (14.7 KB)

		  qbrfefb9298-fa Link encap:Ethernet  HWaddr 86:78:ab:8d:92:57
          inet6 addr: fe80::a841:8eff:fe18:9db5/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:130257 errors:0 dropped:0 overruns:0 frame:0
          TX packets:10976 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:23382621 (23.3 MB)  TX bytes:1696992 (1.6 MB)

qvbada6a901-dc Link encap:Ethernet  HWaddr be:ef:e4:57:96:79
          inet6 addr: fe80::bcef:e4ff:fe57:9679/64 Scope:Link
          UP BROADCAST RUNNING PROMISC MULTICAST  MTU:1500  Metric:1
          RX packets:987 errors:0 dropped:0 overruns:0 frame:0
          TX packets:577 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
		  
          RX bytes:187623 (187.6 KB)  TX bytes:105707 (105.7 KB)
qvoada6a901-dc Link encap:Ethernet  HWaddr 02:a9:ba:2f:ac:f6
          inet6 addr: fe80::a9:baff:fe2f:acf6/64 Scope:Link
          UP BROADCAST RUNNING PROMISC MULTICAST  MTU:1500  Metric:1
          RX packets:577 errors:0 dropped:0 overruns:0 frame:0
          TX packets:987 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
		  
          RX bytes:105707 (105.7 KB)  TX bytes:187623 (187.6 KB)
tapada6a901-dc Link encap:Ethernet  HWaddr fe:16:3e:56:9b:2d
          inet6 addr: fe80::fc16:3eff:fe56:9b2d/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:18 errors:0 dropped:0 overruns:0 frame:0
          TX packets:185 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:500
          RX bytes:1860 (1.8 KB)  TX bytes:24701 (24.7 KB)

qvbfefb9298-fa Link encap:Ethernet  HWaddr 86:78:ab:8d:92:57
          inet6 addr: fe80::8478:abff:fe8d:9257/64 Scope:Link
          UP BROADCAST RUNNING PROMISC MULTICAST  MTU:1500  Metric:1
          RX packets:130268 errors:0 dropped:0 overruns:0 frame:0
          TX packets:144163 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:25207738 (25.2 MB)  TX bytes:27523134 (27.5 MB)

qvofefb9298-fa Link encap:Ethernet  HWaddr 8a:96:db:5e:ea:ce
          inet6 addr: fe80::8896:dbff:fe5e:eace/64 Scope:Link
          UP BROADCAST RUNNING PROMISC MULTICAST  MTU:1500  Metric:1
          RX packets:144163 errors:0 dropped:0 overruns:0 frame:0
          TX packets:130268 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:27523134 (27.5 MB)  TX bytes:25207738 (25.2 MB)
```

### Devstack은 OVS 기반으로 네트워크를 구성하며 다음과 같은 가상 네트워킹 디바이스가 사용된다. 

- 탭디바이스 : tapXXX, 인스턴스에서 가상 네트워크 인터페이스를 구현할 때 사용한다. 
- 리눅스 브릿지 : qbrYYYY, 여러 네트워크 인터페이스를 연결하는 가상 인터페이스다. 
- veth페어 : qvbYYYY, qvoYYYY
- OVS 통합 브릿지 : br-int, 통합 브릿지로서 인스턴스와 DHCP서버, 라우터 등과 같은 여러 네트워크 리소스를 연결하는 가상 스위치로서 핵심역할, 인스턴스를 통합 브릿지에 직접 연결하지 않고 리눅스 브릿지를 통해 간접적으로 연결하는 이유는 뉴트론 시큐리티 그룹을 구현하는 데 핵심이 되는 iptables룰을 OVS 브릿지 포트에 직접 연결될 탭 디바이스에 둘 수 없기 때문이다.
- OVS 패치 포트 : int-br-ethX, phy-br-ethX
- OVS 프로바이더 브릿지 : br-ethX, 물리 네트워크 인터페이스와 OVS브릿지를 연결해준다. OVS 패치 포트에 연결된 가상 패치 케이블을 통해 OVS 통합 브릿지와 연결된다.
- OVS 패치 포트 :리눅스 veth 케이블과 비슷하게 동작하지만 OVS 에 좀 더 최적화된 패치 포트라는 내장 포트 타입을 제공한다. 두 개의 OVS 브릿지를 연결할 때 각 스위치에 맞물리는 양쪽 끝 포트를 패치 포트로 할당해 가상 패치 케이블을 생성한다.
- 물리 인터페이스 : ethX

[![ovs-bridge](https://github.com/leeplay/study/blob/master/etc/neutron-networking.png?raw=true)]()

### DHCP 할당 과정 

- DHCP 네트워크 네임스페이스에서 dnsmasq 프로세스가 구동된다.
- DHCP 클라이언트에서 브로드캐스트 주소로 DHCPDISCOVERY 패킷을 보낸다. 
- DHCP 서버는 요청에 다한 응답으로 DHCPOFFER 패킷을 보낸다. 이 패킷에는 요청을 보낸 인스턴스의 MAC, IP 주소, 서브넷 마스크 임대기간, DHCP서버의 IP 주소가 담겨 있따.
- 응답을 받은 DHCP 클라이언트는 다시 DHCPREQUEST 패킷을 DHCP 서버로 보내어 서버에서 제공할 주소를 요청한다. 오직 한 개만 수락한다.
- 이러한 요청에 대해 DHCP 서버는 DHCPACK 패킷을 인스턴스에게 보낸다. 이때 IP 설정이 완료된다. DHCP 서버는 namesevers나 routes와 같은 다른 DHCP 옵션도 인스턴스에게 보낸다. 


```
root@kyu-HP-EliteBook-2570p:/var/run/netns# ls
qdhcp-59089373-ec4d-44b9-b786-0a4122d36bba  qrouter-cd8aaa3a-7e4c-4fa9-b89d-d0a68aef40d5
```

```
root@kyu-HP-EliteBook-2570p:/var/run/netns# ip netns exec qdhcp-59089373-ec4d-44b9-b786-0a4122d36bba ip link list
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
32: tapdd1b55ca-fd: <BROADCAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT group default
    link/ether fa:16:3e:85:2c:22 brd ff:ff:ff:ff:ff:ff
```

```
root@kyu-HP-EliteBook-2570p:/home/stack/openstack/devstack# ovs-vsctl show
8a3f94fa-2b5f-4fa0-ba1b-c11e4a4eae9e
    Bridge br-ex
        Port br-ex
            Interface br-ex
                type: internal
        Port "qg-b3f5b9d1-5e"
            Interface "qg-b3f5b9d1-5e"
                type: internal
    Bridge br-int
        fail_mode: secure
        Port "qvo78e1e93f-27"
            tag: 1
            Interface "qvo78e1e93f-27"
        Port "tapdd1b55ca-fd"
            tag: 1
            Interface "tapdd1b55ca-fd"
                type: internal
...
```

뉴트론 라우팅
=============

클라우드에서 인스턴스에 대한 IP 라우팅과 NAT서비스를 제공할 수 있다. 네트워크를 생성하고 이를 라우터에 연결시키면 여기에 연결된 인스턴스와 여기서 구동되는 어플리케이션을 인터넷에 연결할 수 있다. 

### 외부 프로바이더 네트워크 생성

- create_network
- create_subnet 
- create_router
- router_set_gateway 

### 내부 네트워크 생성 

### 인스턴스 생성 

```
root@kyu-HP-EliteBook-2570p:/var/run/netns# ls
qdhcp-59089373-ec4d-44b9-b786-0a4122d36bba  qrouter-cd8aaa3a-7e4c-4fa9-b89d-d0a68aef40d5
```

```
root@kyu-HP-EliteBook-2570p:/var/run/netns# ip netns exec qrouter-cd8aaa3a-7e4c-4fa9-b89d-d0a68aef40d5 ip link list

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
30: qr-8d969929-f7: <BROADCAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT group default
    link/ether fa:16:3e:7e:25:eb brd ff:ff:ff:ff:ff:ff
31: qg-b3f5b9d1-5e: <BROADCAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT group default
    link/ether fa:16:3e:72:a5:92 brd ff:ff:ff:ff:ff:ff
```

```
root@kyu-HP-EliteBook-2570p:/home/stack/openstack/devstack# ovs-vsctl show
8a3f94fa-2b5f-4fa0-ba1b-c11e4a4eae9e
    Bridge br-ex
        Port br-ex
            Interface br-ex
                type: internal
        Port "qg-b3f5b9d1-5e"
            Interface "qg-b3f5b9d1-5e"
                type: internal
    Bridge br-int
...
        Port "qr-8d969929-f7"
            tag: 1
            Interface "qr-8d969929-f7"
                type: internal
...
    Bridge br-tun
        Port patch-int
            Interface patch-int
                type: patch
                options: {peer=patch-tun}
        Port br-tun
            Interface br-tun
                type: internal
    ovs_version: "2.0.2"
```

### 로드 밸런싱

### 방화벽 

ML2 plugin : 다음 주 예정 ...
==========
