Network Namespace
=================

- 네트워크 장치, 주소, 포트, 라우트, 방화벽 규칙 등의 사용을 분할하여 인스턴스가 실행 중이 환경에서 네트워크를 가상화함
- 네트워크 가상화의 가장 기본 기술 (OpenStack, Docker에서 사용함)
- 커널 2.6.24 버전에서 추가 

### 실습

- namespace 생성 ( /var/lib/netns 경로에 생성됩니다.)
- neutron (https://github.com/openstack/neutron/blob/592f641b4673b07737d689187bc999dff9e4a502/neutron/agent/linux/ip_lib.py#L130)
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

# route -FC
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         10.64.51.1      0.0.0.0         UG    0      0        0 eth0
10.64.51.0      *               255.255.255.0   U     1      0        0 eth0
172.17.0.0      *               255.255.0.0     U     0      0        0 docker0
192.168.122.0   *               255.255.255.0   U     0      0        0 virbr0

# ip route
default via 10.64.51.1 dev eth0  proto static
10.64.51.0/24 dev eth0  proto kernel  scope link  src 10.64.51.185  metric 1
172.17.0.0/16 dev docker0  proto kernel  scope link  src 172.17.42.1
192.168.122.0/24 dev virbr0  proto kernel  scope link  src 192.168.122.1
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







Neutron 
=======



- 2013.2월에 Havana 버전에서 등장 
- 기존 버전에서 Quantum 이란 명칭이었는데 SDN 개념이 도입되면서 Neutron으로 변경
- Quantum은 IceHouse 버전에서 2014.1월에 drop 됨


ML2 plugin
==========
