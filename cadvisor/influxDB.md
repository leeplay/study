https://github.com/google/cadvisor
https://github.com/influxdb/influxdb

cAdvisor는 수치를 influxDB에 저장하는 것을 지원합니다. 
cAdvisor는 컨테이너의 수치를 promethes metrics로 노출합니다. 
heapster는 cAdvisors를 사용하여 cluster 컨테이너의 모니터링을 가능하게 합니다.  

http://influxdb.com/docs/v0.9/introduction/overview.html
위 페이지 분석 후

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
- 


    SQL-like query language.
    HTTP(S) API for data ingestion and queries.
    Built-in support for other data protocols such as collectd.
    Store billions of data points.
    Tag data for fast and efficient queries.
    Database-managed retention policies for data.
    Built in management interface.
    Aggregate on the fly:

Goal
====


    Stores metrics data (like response times and cpu load. i.e. what you’d put into Graphite).
    Stores events data (like exceptions, user analytics, or business analytics).
    HTTP(S) interface for reading and writing data. Shouldn’t require additional server code to be useful directly from the browser.
    Security model that will enable user facing analytics dashboards connecting directly to the HTTP API.
    Horizontally scalable.
    On disk and in memory. It shouldn’t require a cluster of machines keeping everything in memory since most analytics data is cold most of the time.
    Simple to install and manage. Shouldn’t require setting up external dependencies like Zookeeper and Hadoop.
    Compute percentiles and other functions on the fly.
    Downsample data on different windows of time.
    Can efficiently and automatically clear out raw data daily to free up space. Database managed retention policies.
    Should be able to add on custom real-time and batch analytics processing.
    Expand storage by adding servers to a cluster. Should make node replacement quick and easy.
    Automatically compute common queries continuously in the background.


Getting Started 후

api 사용법 파악 



