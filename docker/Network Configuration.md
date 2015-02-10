TL;DR
=====

도커가 시작될 때 호스트 머신에 docker0이라 불리는 가상 인터페이스를 생성합니다. 
호스트에서 RFC 1918 기준으로 private한 범위에서 랜덤으로 ip 주소와 서브넷를 선택합니다.  
그리고 아이피를 docker0에 할당합니다. 도커가 시작되기 몇 분 전에 도커는 172.17.42.1/16 중에 하나를 선택합니다.  
예를 들어 16 bit netmask는 호스트머신과 컨테이너를 위해 65,534 개의 주소를 제공합니다.
맥 어드레스는 ARP 충돌을 피하고 할당된 컨테이너의 ip어드레스를 사용하기 위해  2 66 172 17 0 0 ~ 2 66 172 17 255 255 범위에서 생성됩니다. 

docker0은 평범한 인터페이스는 아닙니다. 
자동으로 다른 접근 가능한 네트워크 인터페이스와 패킷 포워드를 하기위한 가상 이더넷 브릿지 입니다. 
컨테이너가 호스트 머신, 외부와 통신을 가능케 합니다. 

도커는 매번 컨테이너를 만들 때 한쪽에서 패킷을 보내고 다른 쪽에서 받을 수 있는 파이프와 같은 짝을 이루는 peers 인터페이스를 생성합니다. peer는 vethAQI2QT와 같은 유니크한 이름을 주어 컨테이너 내의 eth0 인터페이스가 되고 다른 피어와 연결을 유지합니다. 모든 veth* 인터페이스는 docker0을 통한 브릿지와 바인딩에 의해 도커는 호스트 머신과 모든 도커 컨테이너와 공유된 가상 subnet을 생성합니다. 

> 서브넷(subnet)은 "subnetwork을 줄인 말로서 어떤 기관에 소속된 네트웍이지만 따로 분리되어 있는 한 부분으로 인식될 수 있는 네트웍을 말한다. 일반적으로 하나의 서브넷은 하나의 지역, 한 빌딩 또는 같은 근거리통신망 내에 있는 모든 컴퓨터들을 나타낼 수 있다. 여러 개의 서브넷으로 나뉘어진 어떤 조직의 네트웍은 인터넷에 하나의 공유된 네트웍 주소로 접속될 수 있다. 만약 서브넷이 없다면, 그 조직은 물리적으로 분리된 서브 네트웍마다 하나씩, 여러 군데의 인터넷 접속을 가지게 될 것이며, 그렇게 함으로써 한정된 량의 인터넷 주소가 쓸모 없이 낭비될 수도 있게된다. 

Configuring DNS
===============
도커는 호스트네임과 DNS 설정을 어떻게 지원할까요? 
최신 정보를 작성할 수 있도록 가상화한 파일을 etc에 오버레이합니다. 실행 중인 컨테이너에서 mount 명령어를 치면 볼 수 있습니다. 

도커는 resolv.conf(네임서버 정보가 담겨있음)를 호스트 머신이 새로운 DHCP 정보를 받을 때 최신 상태로 유지시킵니다. 이 방식에 대한 정보는 도커 다음 버전에서 변경될 수도 있으니 직접 설정 파일을 수정하지 말고 옵션을 통해 사용하도록 합니다. 
- -h, --hostname /etc/hostname과 /etc/hosts에 설정됨, 프롬프트에도 이 이름으로 보임, 하지만 다른 컨테이너나 데몬에서는 볼 수 없음
- --link 이 옵션을 사용하면 /etc/hosts에 ip와 alias 정보가 기록된다. 컨테이너가 재실행하면 link 정보는 변경되지 않는다. 
- --dns 옵션을 사용하면 /etc/resolv.conf의 정보로 기록된다. 
- --dns-search 옵션을 사용하면 dns 정보를 검색해 /etc/resolv.conf에 기록한다. 

Communication between containers and the wider world
====================================================

호스트 머신은 어떻게 ip 패킷 포워드를 할 수 있는가 ?, ip_forward란 리눅스 시스템 파라미터에 의해 결정됩니다. 
만약 파라메터가 1(true)이면 컨테이너들 끼리 패킷을 전달 할 수 있습니다. 보통 Docker 서버를 떠나 true로 설정하고 사용할 것이며 도커 데몬 시작 시에 도커에 의해 1로 설정됩니다. 

Communication between containers
================================

- 네트워크 토폴로지에도 컨테이너의 네트워크 인터페이스를 연결합니까? 디폴트로 도커는 모든 컨테이너를 패킷을 보내기 위해 docker0 브릿지로 연결합니다. 
- iptables는 이 특별한 연결을 어떻게 생성할까요 ? 도커는 데몬이 시작할 때 iptables=false라면 시스템의 iptables를 변경하지 않습니다. 반면에 도커 서버는 포워드 체인에 icc가 true이면 ACCEPT를 false이면 DROP 룰을 추가합니다.  

> 토폴로지(topology, 문화어: 망구성방식)는 컴퓨터 네트워크의 요소들(링크, 노드 등)을 물리적으로 연결해 놓은 것, 또는 그 연결 방식을 말한다.

icc가 true 혹은 false 이든 이것은 전략적인 질문입니다. iptables은 접근되는 임의의 포트로부터 도커 호스트와 컨테이너들을 보호합니다. 가장 보안적인 설정인 icc=false를 선택했다면, 이 경우 다른 서비스를 제공하기 위해 컨테이너 통신은 어떻게 할 수 있을까요 ?

이 질문에 답은 이전 섹션에 언급되어진 --link옵션입니다. 만약 icc=false, iptables=true로 도커 데몬이 실행 중에 도커 컨테이너를 --link옵션으로 실행했다면 도커 서버는 iptables의 ACCEPT룰을 입력하고 새로운 컨테이너가 노출된 다른 컨테이너의 노출된 포트로 접속을 할 수 있도록 합니다.

도커 호스트에서 iptables 커맨드를 통해 포워드 체인이 가진 ACCEPT, DROP 규칙의 확인이 가능하다.

```
# When --icc=false, you should see a DROP rule:

$ sudo iptables -L -n
...
Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
DROP       all  --  0.0.0.0/0            0.0.0.0/0
...

# When a --link= has been created under --icc=false,
# you should see port-specific ACCEPT rules overriding
# the subsequent DROP policy for all other packets:

$ sudo iptables -L -n
...
Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
ACCEPT     tcp  --  172.17.0.2           172.17.0.3           tcp spt:80
ACCEPT     tcp  --  172.17.0.3           172.17.0.2           tcp dpt:80
DROP       all  --  0.0.0.0/0            0.0.0.0/0
```

Binding container ports to the host
===================================

도커 컨테이너는 외부와 연결할 수 있습니다. 그러나 외부는 컨테이너로 연결할 수 없습니다. 외부로의 연결은 iptables의 masquerading 룰로 인해(가상 아이피로 외부와 통신이 가능하게 하는 옵션 룰) 가능합니다. 

```
# You can see that the Docker server creates a
# masquerade rule that let containers connect
# to IP addresses in the outside world:

$ sudo iptables -t nat -L -n
...
Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
MASQUERADE  all  --  172.17.0.0/16       !172.17.0.0/16
...
```

도커 컨테이너로 연결을 원한다면 docker run 실행 시에 특별한 옵션을 주어야 합니다.(자세한 건 사용 guide에서 확인하십시오) 두 가지 접근 방법이 있습니다. 

- -P(대문자) or --publis-all=true|false : Dockerfile에서 EXPOSE 명령어를 통해 설정한 모든 포트를 오픈하도록 해준다.
- -p(소문자) or --publish :  도커가 실행 중에 어떤 네트워크 포트를 오픈할지를 관리한다

어떤 방법을 택하던지 간에 iptables의 DNAT 항목으로 확인이 가능해야 합니다. 

```
# What your NAT rules might look like when Docker
# is finished setting up a -P forward:

$ iptables -t nat -L -n
...
Chain DOCKER (2 references)
target     prot opt source               destination
DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:49153 to:172.17.0.2:80

# What your NAT rules might look like when Docker
# is finished setting up a -p 80:80 forward:

Chain DOCKER (2 references)
target     prot opt source               destination
DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:80 to:172.17.0.2:80
```

항상 정해진 아이피에 바인딩을 해야한다면 /etc/default/docker의 DOCKER_OPTS 항목에 --ip=IP_ADDRESS 옵션으로 설정이 가능합니다. 

Customizing docker0
===================

docker는 싱글 이더넷 네트워크처럼 작동하는 물리 혹은 가상 네트워크 인터페이스 사이로 패킷을 주고 받을 수 있도록 호스트 시스템의 리눅스 커널안에 docker0 이더넷 브릿지를 생성합니다. 

docker0에 ip address와 netmaskk, ip 할당 범위를 설정할 수 있습니다. 호스트 머신은 다른 브릿지를 통해 연결된 컨테이너에 패킷을 송수신 할 수 있습니다. 

- -bip=CIDR docker0 브리지에 특정한 ip와 netmask를 할당합니다. 
- --fixed-cidr docker0 subnet에 ip 범위를 제한을 줄 수 있습니다. 
- --mtu maximum transmission unit 의 약자로 docker0이 허용하는 최대 패킷 길이를 정의합니다.

docker0 브리지에 연결된 컨테이너를  brctl 명령으로 확인가능합니다.

```
# Display bridge info

$ sudo brctl show
bridge name     bridge id               STP enabled     interfaces
docker0         8000.3a1d7362b4ee       no              veth65f9
                                                        vethdda6
```
brctl이 설치 안되어있다면 설치하세요 
sudo apt-get install bridge-utils

```
# The network, as seen from a container

$ sudo docker run -i -t --rm base /bin/bash

$$ ip addr show eth0
24: eth0: <BROADCAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 32:6f:e0:35:57:91 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.3/16 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::306f:e0ff:fe35:5791/64 scope link
       valid_lft forever preferred_lft forever

$$ ip route
default via 172.17.42.1 dev eth0
172.17.0.0/16 dev eth0  proto kernel  scope link  src 172.17.0.3

$$ exit
```
ip route로 확인해보면 컨테이너의 eth0이 docker0과 연결된 걸 확인할 수 있습니다. 
그리고 명심하세요 도커는 리눅스 시스템에 ip_forward가 fasle면 패킷을 존송하지 않습니다. 

Building your own bridge
========================

docker 시작 전에 -b or --bridge 옵션을 통해서 도커가 제공하는 docker0 말고 직접 설정한 브릿지를 사용할 수 있습니다. 그리고 docker가 실행된 후에 docker0 브릿지의 정보를 수정하려면 docker0을 down 한 후 interface에서 제거 후 설정해야 합니다. 아래는 설정 방법입니다. 

```
# Create our own bridge

$ sudo brctl addbr bridge0
$ sudo ip addr add 192.168.5.1/24 dev bridge0
$ sudo ip link set dev bridge0 up

# Confirming that our bridge is up and running

$ ip addr show bridge0
4: bridge0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state UP group default
    link/ether 66:38:d0:0d:76:18 brd ff:ff:ff:ff:ff:ff
    inet 192.168.5.1/24 scope global bridge0
       valid_lft forever preferred_lft forever

# Tell Docker about it and restart (on Ubuntu)

$ echo 'DOCKER_OPTS="-b=bridge0"' >> /etc/default/docker
$ sudo service docker start

# Confirming new outgoing NAT masquerade is set up

$ sudo iptables -t nat -L -n
...
Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
MASQUERADE  all  --  192.168.5.0/24      0.0.0.0/0
```

How Docker networks a container
===============================

도커는 네트워크 설정은 계속 개발 중이고 변경 중입니다. 이번 섹션의 쉘 커맨드는 도커가 새로운 컨테이너를 만들 때 수행하는 네트워크 설정 작업을 러프한 레벨에서 단면적인 단계로 나타냅니다.

도커는 호스트 머신과 컨테이너와의 통신을 위해 호스트 머신 안에 패킷을 보낼 수 있도록 링킹된 peers라고 불리는 특별한 가상 인터페이스를 사용합니다.

- 짝을 이루는 peer 가상 인터페이스 생성
- veth65f9와 같은 유일한 이름을 부여받음, docker 호스트에 유지되고 있다가 docker0에 바인드 됨 
- 다른 인터페이스는 컨테이너 내에서 분리되고 유일한 네트워크 인터페이스 네임스페이스를 가진 상태로 eth0이란 이름으로 새로운 컨테이너로 전달합니다. 실제 물리 네트워크 인터페이스가 아닙니다. 
- 인터페이스에 랜덤 or --mac-address 인자로 받은 맥 어드레스를 설정합니다. 
- 브릿지의 범위안의 ip 를 컨테이너의 eth0에 부여합니다. ip 주소는 docker 호스트와 브릿지 됩니다. 

위의 스텝이 성공하면 컨테이너는 eth0 으로 다른 컨테이너 혹은 인터넷이 가능해집니다. 
위의 절차를 따르지 않고 --net 옵션을 통해 docker 컨테이너 실행 시 특별한 옵션을 줄 수 있습니다. 

- --net=bridge 기본 설정, 위의 절차대로 작동함 
- --net=host
- --net=container:NAME_or_ID
- --net=none

Tools and Examples
====================

네트워크 토폴리지 구성을 보기 전에 흥미를 끌만한 몇 가지 툴과 예를 안내합니다. 

- Jérôme Petazzoni has created a pipework shell script to help you connect together containers in arbitrarily complex scenarios: https://github.com/jpetazzo/pipework
 
- Brandon Rhodes has created a whole network topology of Docker containers for the next edition of Foundations of Python Network Programming that includes routing, NAT'd firewalls, and servers that offer HTTP, SMTP, POP, IMAP, Telnet, SSH, and FTP: https://github.com/brandon-rhodes/fopnp/tree/m/playground

Building a point-to-point connection
====================================

특정 컨테이너 둘이 복잡한 설정없이 바로 통신을 하길 원할 수 있습니다. 
이 해결책은 간단합니다. simply throw both of them into containers, and configure them as classic point-to-point links. The two containers will then be able to communicate directly (provided you manage to tell each container the other's IP address, of course). You might adjust the instructions of the previous section to go something like this:

```
# Start up two containers in two terminal windows

$ sudo docker run -i -t --rm --net=none base /bin/bash
root@1f1f4c1f931a:/#

$ sudo docker run -i -t --rm --net=none base /bin/bash
root@12e343489d2f:/#

# Learn the container process IDs
# and create their namespace entries

$ sudo docker inspect -f '{{.State.Pid}}' 1f1f4c1f931a
2989
$ sudo docker inspect -f '{{.State.Pid}}' 12e343489d2f
3004
$ sudo mkdir -p /var/run/netns
$ sudo ln -s /proc/2989/ns/net /var/run/netns/2989
$ sudo ln -s /proc/3004/ns/net /var/run/netns/3004

# Create the "peer" interfaces and hand them out

$ sudo ip link add A type veth peer name B

$ sudo ip link set A netns 2989
$ sudo ip netns exec 2989 ip addr add 10.1.1.1/32 dev A
$ sudo ip netns exec 2989 ip link set A up
$ sudo ip netns exec 2989 ip route add 10.1.1.2/32 dev A

$ sudo ip link set B netns 3004
$ sudo ip netns exec 3004 ip addr add 10.1.1.2/32 dev B
$ sudo ip netns exec 3004 ip link set B up
$ sudo ip netns exec 3004 ip route add 10.1.1.1/32 dev B
```

The two containers should now be able to ping each other and make connections successfully. Point-to-point links like this do not depend on a subnet nor a netmask, but on the bare assertion made by ip route that some other single IP address is connected to a particular network interface.

Note that point-to-point links can be safely combined with other kinds of network connectivity — there is no need to start the containers with --net=none if you want point-to-point links to be an addition to the container's normal networking instead of a replacement.

A final permutation of this pattern is to create the point-to-point link between the Docker host and one container, which would allow the host to communicate with that one container on some single IP address and thus communicate “out-of-band” of the bridge that connects the other, more usual containers. But unless you have very specific networking needs that drive you to such a solution, it is probably far preferable to use --icc=false to lock down inter-container communication, as we explored earlier.

Editing networking config files
===============================

Docker 1.2.0 에서부터 컨테이너 안에 /etc/hosts, /etc/hostname, /etc/resolve.conf 를 수정할 수 있습니다. 이 기능은 유용하게 사용할 수 있지만 docker commit과 docker run 시에 저장되지 않습니다. 이미지에 저장되지 않는다는 말입니다. they will only "stick" in a running container.( 관용어 같은데 해석이 안됨)
