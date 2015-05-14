Overview 
========

- go 로 만들어졌고 외부 의존성이 없음 
- DevOps, Metrics, Sensor data, real-time 분석이 타겟
- v0.9.0-rc30 (릴리즈 주기가 1주일 이내일 정도로 업데이트가 빠름)

Key Features
=============

- SQL 쿼리 랭귀지
- HTTP(S) API 지원
- collectd 같은 데이터 프로토콜 지원 
- 수억개의 데이터 포인트 저장 (기선님께 확인해보자)
- 빠르고 효율적인 쿼리를 위한 데이터 태그
- 즉시 집계 
- 내장된 관리 인터페이스
- data를 위한 database 관리 보유 정책 

Goal
====

- metrics data 저장 (e.g. cpu, 응답시간)
- events data 저장 (e.g. 예외, 사용자, 비즈니스 분석)
- data reading, writing HTTP(S) api 지원
- dashboard 
- Horizontally scalable (i.g. 수평적 구조로 용량이 부족하면 병렬적으로 추가)
- 간단한 설치와 관리 
- on disk, in memory
- downsample data
- 백분위 계산 
- daily raw data를 clear out 기능
- 사용자 정의 실시간 분석 배치 작업 추가 가능 
- 손쉬운 노드 교체로 스토리지 확장 
- 자동계산에 의해 지속적인 공통 쿼리의 백그라운 수행 지원
    
Getting Started
================



api 사용법 파악 

출처 : http://influxdb.com/docs/v0.9/introduction/overview.html

