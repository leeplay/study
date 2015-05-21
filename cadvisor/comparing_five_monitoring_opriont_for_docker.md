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

The next approach for docker monitoring is Scout and it addresses several of limitations of CAdvisor. Scout is a hosted monitoring service which can aggregate metrics from many hosts and containers and present the data over longer time-scales. It can also create alerts based on those metrics. The first step  to getting scout running is to sign up for a Scout account at https://scoutapp.com/, the free trial account should be suitable for testing out integration.  Once you have created your account and logged in, click on your account name in the top right corner and then Account Basics and take note of your Account Key as you will need this to send metrics from our docker server.



## Data Dog

## Sensu

## Score Card

- 5점 만점

|          |docker|cadvisor|scout|Data Dog|Sensu|
|----------|------|--------|-----|--------|-----|
|쉬운 배포|5|5|4|5|1|
|detail 레벨|5|2|2|5|4|
|aggregation 레벨|0|1|3|5|4|
|alert 지원|0|0|3|Supported|Supported but limited|
|non-docker 리소스 지원|X|X|O|5|5|
|비용|free|free|$10/host|15/host|Free|

## conclusion



## 출처

- http://rancher.com/comparing-monitoring-options-for-docker-deployments/
