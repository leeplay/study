## containerD: 살아남는 종은 강한 종이 아니고 또 똑똑한 종도 아니다. 변화에 적응하는 종이다.


### overview

- docker engine에서 파생
- 2015년 11월 runc를 제어하기 위해 시작
- docker 1.11 version에서 사용됨 
- 2016년 12월에 새로운 영역으로 재시작
- 2019년 2월 28일 cncf 공식 프로젝트로 발표됨
- 현재 docker는 default/best 실행환경이며 이런 의존을 대체하기 위해 시작
- linux와 window를 지원하기 위한 데몬
- containerd와 runc는 오픈소스 프로젝트로 cncf/linux에서 인큐베이팅

[![containerD](https://github.com/leeplay/study/blob/master/image/cd_image04.png?raw=true)]()


### promise of containerD

- container 실행 관리
- 이미지 배포
- 네트워크 네임스페이스 관리
- 로컬 스토리지 접근
- native plumbing 수준의 API 지원
- gPRC-based API
- Full OCI supoort
- OCI 런타임 제공 (aka runC)
- decoupled system (image, filesystem, runtime)
- 안전/견고/성능이 우수한 코어 컨테이너 런타임
- 이 모든걸 window/linux 동일하게 지원
- 완벽한(이미지 전송/저장, 저수준 스토리지 관리, 네트워크 접근) 컨테이너 라이프사이클 관리


### layer

- 새로운 아키텍처 소개
  - Docker Engine v1.11
  - 도커 엔진의 컨테이너 실행 환경의 일부분을 별도 데몬으로 재구성
- containerD : 컨테이너 라이프사이클 관리를 위한 경량 데몬, core container runtime
- container-shim : 컨테이너와 containerd 사이에서 컨테이너를 연결, containerd가 문제가 생기더라도 실행되는 컨테이너에는 영향을 주지않기 위한 레이어
- runc : OCI를 준수하는 리눅스 컨테이너 실행환경
  


[![containerD](https://github.com/leeplay/study/blob/master/image/cd_image05.png?raw=true)]()


- FOSS stack
  - Commercial       <-> Docker Enterprise
  - Differentiators  <-> API/CLI/Compose/Build/Auth/Trust/Distribution
  - Orchestration    <-> SwarmKit
  - Runtime          <-> containerD
  - Infrastructure   <-> InfraKit


[![containerD](https://github.com/leeplay/study/blob/master/image/cd_image07.png?raw=true)]()

### architecture in window/linux

- linux
  - docker -> containerd + runc

[![containerD](https://github.com/leeplay/study/blob/master/image/cd_image01.png?raw=true)]()

- window (as-is)
  - docker -> HCS(host compute service)
  - window는 hcs가 os에 내장되어 있고 docker에서 직접 접근하도록 열어준 거 같음

[![containerD](https://github.com/leeplay/study/blob/master/image/cd_image02.png?raw=true)]()

- window (to-be)
  - docker -> containerd + runhsc
  - linux 아키텍처와 동일해짐
  - window server 2019/window 10 1809 부터 cri 지원한다 함
  
- 실제론 이렇게 쓰일 듯 ?
  - kube -> cri -> containerd + (runc/runchcs) -> core platform


### in kube

- kube runtime을 위해 CRI를 사용
- kubelet -> cri -> containerD -> container
- 설계

[![containerD](https://github.com/leeplay/study/blob/master/image/cd_image08.png?raw=true)]()

- 구현
[![containerD](https://github.com/leeplay/study/blob/master/image/cd_image09.png?raw=true)]()



### utopia


- orchestration 기준

[![containerD](https://github.com/leeplay/study/blob/master/image/cd_image10.png?raw=true)]()

- os 포함

[![containerD](https://github.com/leeplay/study/blob/master/image/cd_image06.png?raw=true)]()


### conclusion 

- 어차피 생태계는 다양하고 이 다양한 생태계에 범용적인 것을 만들고 규격화하기 위한 오픈소스 진영의 노력이지만 실제 docker/google이 그리는 그림
- docker로 접하는 컨테이너 기술들이 점점 시장참여자들의 주도권 다툼으로 파편화되가는 느낌이며 서로 시장을 뺏기지 않기위해 서로가 서로를 지원해가며 복잡하고 다양한 아키텍처를 그려나가는 거 같으며 좀 더 우위를 점하는 부분들이 나타나기도 하지만 겹치는 부분들도 많이 보임
- 2018년도에 활발하게 개발 중, 즉 격변기
- <h3>살아남는 종은 강한 종이 아니고 또 똑똑한 종도 아니다. 변화에 적응하는 종이다.</h3>

[![containerD](https://github.com/leeplay/study/blob/master/image/cd_image11.png?raw=true)]()






















