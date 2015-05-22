## Comparing Five Monitoring Options for Docker

## Docker Stats

- docker 1.5.0에서 컨테이너의 리소스 사용량을 알려주는 커맨드 추가
- 각 컨테이너의 cpu 이용량, 사용 중인 메모리와 전체 메모리, 네트워크 전송량 정보를 제공
- docker remote api를 직접 개발해 더 자세한 stat 정보를 얻을 수 있음

```
$ docker stats determined_shockley determined_wozniak prickly_hypatia
CONTAINER             CPU %               MEM USAGE/LIMIT       MEM %               NET I/O
determined_shockley   0.00%               884 KiB/1.961 GiB     0.04%               648 B/648 B
determined_wozniak    0.00%               1.723 MiB/1.961 GiB   0.09%               1.266 KiB/648 B
prickly_hypatia       0.00%               740 KiB/1.961 GiB     0.04%               1.898 KiB/648 B
```

## CAdvisor

- docker stats, remote API 커맨드 라인에서 정보를 얻기에는 유용하지만 그래프 형태로 보기에는 적합하지 않음
- CAdvisor는 docker stat에 의한 정보를 비주얼 하게 제공 
- cpu, memory, network, disk-space 리소스를 그래프로 확인 가능
- 설치와 사용이 간편함 
- aggregation 지원
- 오픈소스이며 클러스터에서 provisioned 되어 동작함 
- 하나의 docker host만 모니터 가능
- 쿠베 환경의 멀티 노드를 모니터 하려면 heapster를 사용해야 함.
- alerting 메커니즘이 없음 
- rancher에서 cAdvisor를 사용 

## Scout

Scout는 많은 호스트와 컨테이너 그리고 오랜 시간에 걸친 데이터의 metrics를 aggregate 할 수 있는 hosted monitoring 서비스 입니다. metrics 기반으로 alert도 만들 수 있습니다. 

- scout계정을 생성 후 accout_key를 생성함 
- 호스트에 scoutd.yml 파일을 생성 후 accout_key와 모니터링 대상의 정보를 작성
- scout agent 컨테이너를 실행
- scout web view에서 모니터링이 되는 걸 확인
- trigger 설정 

Another advantage of using Scout over CAdvisor is that it has a large set of plugins which can pull in other data about your deployment in addition to docker information. This allows Scout to be your one stop monitoring system instead of having a different monitoring system for various resources in your system.

One drawback of Scout is that it does not present detailed information about individual containers on each host like CAdvisor can. This is problematic, if your are running heterogeneous containers on the same server. For example if you want a trigger to alert you about issues in your web containers but not about your Jenkins containers Scout will not be able to support that use case. Despite the drawbacks Scout is a significantly more useful tool for monitoring your docker deployments. However this does come at a cost, ten dollars per monitored host. The cost could be a factor if you are running a large deployment with many hosts.


## Data Dog

## Sensu

## Score Card

- 5점 만점

|              |docker|cadvisor|scout|Data Dog|Sensu|
|--------------|------|--------|-----|--------|-----|
|쉬운 배포     |5     |5       |4    |5       |1    |
|detail 레벨   |5     |2       |2    |5       |4    |
|aggregation   |none  |1       |3    |5       |4    |
|alert 지원    |none  |none    |3    |Supported|Supported but limited|
|non-docker지원|none  |none    |Supported|5   |5    |
|비용          |free  |free    |$10/host|15/host|Free|

## conclusion



## 출처

- http://rancher.com/comparing-monitoring-options-for-docker-deployments/
