## container

- lxc
  - resource isolation
  - cgroups
    - cpu, memory, block io, file system
  - namespace
    - 격리화된 network interface를 제공하는 기술
  - chroot
    - 프로세스가 인식하는 루트 디렉터리를 변경
  - union mount
    - 동일한 디렉토리에 여러 파일시스템을 마운트하는 기술·기능
    - 먼저 마운트된 것을 살려둔 상태로 추가적으로 마운트하는 것/단, 겹치는 것(폴더, 파일)이 있으면 나중 것이 우선함
- 이것들을 패키징하고 사용성을 높인 대표적 유틸리티가 docker
- etc
  - mac/window/linux -> hypervisor -> hardware
  - 전가상화 
    - cpu의 vt(virtual technology)를 사용
    - 오버헤드가 큼
    - VMware
  - 반가상화
    - 게스트 OS를 수정하여 게스트 OS가 가상화되고 있음을 인식하도록해 하이퍼콜을 최소화함
    - 성능이 향상되지만 구현/관리가 복잡함
    - Xen
  - KVM
    - Kernel-based virtual machine
    - mainline에 포함된 정식 커널 모듈

[![rkt1](https://github.com/leeplay/study/blob/master/image/rkt_image01.png?raw=true)]()


## rkt


### overview

- 2014년 12월 made by coreos
- rock it
- secured, lightweight container runtime
- better than docker
- pod-native approach (= kube orchestration)
- isolation parameters (pod/applcation level)
- support docker image
- Specialized, trusted processes can run like a traditional chroot.
- Normal namespacing and cgroup isolation enforced by software above a shared kernel.
- 기존 docker 이미지를 사용하세요, 거기에 실용적인 빌트인 보안을 제공할게요


### round 1 : rkt vs docker

- Runs Docker images : yes vs yes
- Image Signing : yes(의무) vs yes (의무 아님)
- Privilege Separation (권한분리) : Fetch/verify/validate vs root
- Composability : Proper unix process model vs custom in-container init system
- Image Creation : shell scripting, leveraging familiar unix tools vs dockerfile, build by docker daemon
- Container Distribution : tarballs over HTTPS vs docker registry

[![rkt2](https://github.com/leeplay/study/blob/master/image/rkt_image02.png?raw=true)]()




### round 2 : rkt vs docker

- rkt는 보안중심의 경량 컨테이너 솔루션 vs docker는 엔터프라이즈 컨테이너 오케스트레이션/응용 프로그램 관리/엔터프라이즈 등급의 보안 제공
- 툴 사용성 동급
- 강력한 커뮤니티 보유
  - active hub vs portal, forum (개발자 시각에서는 forum 이 강력한 커뮤니티)
- 정기 릴리즈 (둘 다 1.0 이상, docker가 3년 빠름)
- 과금 정책
  - 둘 다 무료 버전 지원
  - docker datacenter, repo, cloud vs support kube
  - docker는 솔루션급 서비스, 둘 다 비쌈, 내 돈아니면 docker
- API 지원
- 3rd party integration 은 docker 압승 (dokcer hub에서 10만개 프리앱 지원)
- ADP/PayPal/Ebay/BBC/Spotify/Lyft/Expedia/Groupon/GE/ING/Uber vs CA Technologies/Verizon/Viacom/DigitalOcean/Salesforce
- learning curve 둘 다 비슷

[![rkt3](https://github.com/leeplay/study/blob/master/image/rkt_image03.png?raw=true)]()
[![rkt4](https://github.com/leeplay/study/blob/master/image/rkt_image04.png?raw=true)]()

### conclusion

- 홈페이지/깃헙만 보면 rkt <<<<<<<<<<<<<<<<<<<<< 넘을 수 없는 4차원의 벽 <<<<<<<<<<< docker 임 (vs를 하려했는데 포기함)
- rkt는 docker image를 직접 자사환경에서 지원하거나 컨버팅(ACI)을 지원함
- rkt는 컨트리뷰션 상위 랭크 100 에 중국인이 없음
- (docker 는 cli/ser 구조이며, daemon 종료 시 사이드이펙트...) 수많은 이유가 있지만 결론은 복잡한 docker 보다 낫다는 주장
- 하지만 복잡함과 불편함은 어디에 가치를 두냐에 따른 상대적인 것이며 docker vs rkt 이렇게 보면 rkt의 주장이 일리가 있지만 보통 docker는 kube or swarm과 함께 쓰이고 편의/안전성을 어디에서 지원하느냐의 차이인 거 같음 (docker 홈페이지 가면 docker도 쉽고 안전하고 견고하다고 홍보함)
- rkt의 장점이 소프트웨어 개발자가 고려할 수준을 넘어선거 같음

- 그래서 docker 써 rkt 써 ?
  -> 응 난 kube에선 cri-o 써
- 위의 비교들은 현재 폭발적으로 성장하는 오픈소스들에게는 반년만 지나면 대부분 맞지 않는 말일 것이다. (몇 개의 과거 블로그 글들을 보면 장단점으로 꼽았던 것들이 현재는 의미가 없어져있었다.) 그러니 너무 디테일하게 보진 말자, 종합적인 시각에서 kube(main actor)와 메인개발사(google, ms, docker)에 집중하자.
- 향후에 우린 cri 인터페이스(docker, cri-containered, rkt, cri-o, frackti)로 사용하게 될 거 같다.
- GoodBye docker, Hello containers.
