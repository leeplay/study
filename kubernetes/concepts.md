
개념
=========

- 구글에서 공개한 리눅스 컨테이너 관리 시스템
- GCE, AWS 같은 클라우드 환경과 물리 장비를 모두 지원


### Container 

- 컨테이너형 가상화 기술에서 최소 단위

### Pod

- Kubernetes 최소 실행 단위
- Pod 안에서는 같은 네트워크, 디스크 환경을 공유
- YAML/JSON 파일로 선언

### Minion

- 컨테이너가 실행되는 물리적(논리적) 단위
- Node 위의 Docker Daemon에서 컨테이너를 실행
- Kubelet, Proxy

### Kubelet

- 각 Minion에 설치되는 데몬
- 컨테이너와 Pod을 관리하는 역할
- etcd
-cAdvisor

### Label

- 키, 값으로 구성된 메타정보
- Pod/Service/Controller에는 Label을 붙일 수 있다. 
- 태그 역할
- 검색/조작 가능

### Replication Controller 

- 지정한 수만큼 Pod을 실행하도록 해주는 컨트롤러
- 차이가 나면 자동적으로 Pod실행/종료
- 현재는 Kubernetes에서 지원하는 유일한 컨트롤러 

### Service

- 같은 역할을 하는 Pod들을 묶는 단위
- Service가 요청을 받으면 Pod들에 처리를 넘겨줌 


### 출처 

http://www.slideshare.net/ext/devfair-kubernetes-101?qid=96c5fcf4-ab46-4eaf-9709-8c5007f0fd8c&v=qf1&b=&from_search=1
http://www.slideshare.net/naver-labs/docker-kubernetes?qid=891ca20f-7c00-4b81-bf40-0077b2067d64&v=qf1&b=&from_search=2

