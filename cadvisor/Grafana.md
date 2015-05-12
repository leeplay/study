Grafana
=======

cAdvisor는 중요한 데이터를 제공해주지만 데이터를 db에 저장하진 않습니다. Grafana와 InfluxDB로 데이터 저장 및 시각화를 하려 합니다. 

### Graphite

- 시계열 데이터 수집에 최적화된 타임시리즈 데이터베이스 일종 
- 네임스페이스에 시간과 데이터를 계속해서 쌓아가는 특수한 데이터 저장소 
- Graphite-Web 모듈에서 API 형태로 그래프 파일과 수치 데이터를 제공 
- Dashborad 툴로 Grafana가 있음 

### Architecture 

![grafana](https://github.com/leeplay/study/blob/master/etc/graphite.png)

- Collector : Graphite에 어떠한 데이터를 쌓기 위한 모듈
- Carbon-Cache : Collector가 보내온 데이터(네임스페이스, 시간, 데이터)를 받아 Whisper에 저장, 데이터 수집기 
- Whisper : 실제로 데이터를 파일시스템에 저장하고 읽어오는 모듈
- Graphite-Web : http를 통해 Whisper에 저장된 데이터를 가져옴 
