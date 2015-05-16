
## Overview 

- self-hosted tool
- metrics storage, aggregation, visualization, alerting 제공
- 대부분의 툴은 push 기반인데 prometheus는 pull 기반임, custom exporter를 만들 수 있음  
- cadvisor 에 metrics exporter 기능으로 추가되고 있음, 컨트리뷰션 하는 사람이 prometheus 개발자
- influxDB 만큼 인기가 많음, 하지만 아직 둘 다 알파 버전, 단순 시계열 DB 이상의 서비스를 제공함

## Usage

- (대시보드)(http://10.64.51.185:9090/)
- (exporter)(http://10.64.51.185:9104/)
