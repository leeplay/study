 
우리의 목적은 컨테이너, VM, 반가상화 워크로드를 위해 공통 가상 네트워킹 인프라 기반을 제공하는 것이다.
우리는 컴퓨터를 이용하는 복잡한 환경을 지원하기 위해 통합된 가상 네트워크가 필요하다고 생각한다.
어플리케이션 워크로드는 각각의 도달하는 것과 그들이 필요한 인프라 기반 서비스 할 수 있다. 

우리는 컨테이너를 위해 이미 가상화 반가상화를 지원하는 SDN을 주목하기로 했다.
현재 네트워킹 솔루션은 x86 가상화 초창기 상태가 연상되는 컨테이너에 사용되고 있다.

현재 컨테이너를 위한 네트워킹은 제약이 크고 느리다. 
지난 한 해 동안 가상화 머신을 위한 네트워킹은 엄청나게 발전하였다.
그리고 현재 하이퍼바이저는 정기적으로 네트워크 가상화 등의 기술을 포함한 고급 네트워킹 기능을 지원한다.

우리는 진보한 네트워크 가상화를 통해 컨테이너가 더 발전할 것이라고 생각합니다.

도커 컨테이너는 리눅스 환경에 호스팅 됩니다. 
당신은 도커를 지원하는 수많은 리눅스 배포판을 선택할 수 있습니다.
우리는 도커 컨테이너 네트워킹를 두 가지 관점으로 지켜볼겁니다.

- 컨테이너 - 컨테이너, 호스트 안에서 가상 브릿지를 사용
- 컨테이너 - 외부세계 

디폴트로 각 컨테이너의 가상 네트워크 인터페이스는 RFC1918에 정의된 도커가 호스트된 서버에서 사용하지 않는 
프라이빗 아이피 어드레스를 부여받습니다. 

도커 데몬은 docker0이라 불리는 가상 이더넷 브릿지를 생성합니다. 
서버 안에 모든 도커 컨테이너는 이걸 통해 접속을 받습니다.

동일한 서버내에 컨테이너 간 통신은 docker0 브릿지를 통해 이루어집니다. 
브릿지는 트래픽을 모든 인터페이스로 포워드합니다.

새로운 컨테이너가 생성이 될 때 도커는 두 개의 가상 인터페이스를 생성합니다. 
컨테이너와 도커에 의해 서브넷 레인지로부터 할당된 아이피 어드레스로 eth0은 연결됩니다.
디폴트 라우트는 eth0으로 인터페이스 포인트를  docker0 브릿지의 도커 호스트의 아이피 어드레스로 연결됩니다. 

The other interface uses the namespace of the host; it typically has a name of the type veth9a7d and is bound to the docker0 bridge. Docker creates a logical link between the two interfaces. This allows the newly created container to communicate with other containers in the same host and the host itself.

In addition to creating two interfaces for the container, Docker also modifies the host’s iptables so that outbound communication is achieved with IP network access translation (NAT), which modifies iptables to allow outbound traffic and NATs container IP addresses to docker0 IP address.
