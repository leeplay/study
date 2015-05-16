갑자기 prometheus를 보게된 이유는 cAdvisor에서 클러스터 모니터링으로 influxDB, Heapster와 함께 Prometheus를 소개하고 있으며 신기하게도 cAdvisor의 Prometheus를 proposal한 사람이 Docker Inc 개발자였습니다. 게다가 Rancher에서도 Prometheus를 컨테이너 모니터링의 주요 도구로 소개하고 있어 한번 조사를 해봤습니다. 

## Overview 

- self-hosted tool
- metrics storage, aggregation, visualization, alerting 제공
- 대부분의 툴은 push 기반인데 prometheus는 pull 기반임, custom exporter를 만들 수 있음  
- cadvisor 에 metrics exporter 기능으로 추가되고 있음, 컨트리뷰션 하는 사람이 prometheus 개발자
- 단순 시계열 DB 이상의 서비스를 제공하며 influxDB 만큼 인기가 많음, 하지만 아직 둘 다 알파 버전
- rancher 측에서는 cAdvisor를 통한 metrics 데이터를 분석하는 부분에서 prometheus 더 우수하다는 의견입니다.

## Demo

- [services](http://10.64.51.185:9090/)
- [exporter](http://10.64.51.185:9104/metrics)
- [dashboard](http://10.64.51.185:3000/dash1)

## Feature

- cusotm exporter 를 손쉽게 만들 수 있어 모니터링 데이터에 제약이 없다. 
- 주요 storage나 cluster에서 [prometheus exporter](http://prometheus.io/docs/instrumenting/exporters/)를 지원한다.
- 여러 종류의 [시각화 도구](http://prometheus.io/docs/visualization/promdash/)를 붙일 수 있다.  
- notification

## Conclusion 

- 사내 도커 빌드풀에 prometheus를 설치해보는 것도 괜찮을 거 같다.
- 현재 사용 중인 influxDB + cAdvisor + grafana와 비교해보자  
인
## Reference 

- http://rancher.com/comparing-monitoring-options-for-docker-deployments/
- http://rancher.com/docker-monitoring-continued-prometheus-and-sysdig/
