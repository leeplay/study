## Vitess 소개

### Overview

- RDBMS + NoSQL = NewSQL
- ScaleOut을 제공하는 DBMS
- Cloud Native DB
- 무한 Scalable, HA, HS, Reliable 가능
- 사내에서는 n2c, n3r 환경에서 제공
- 20여개 리얼서비스 제공
- Instant Provisioning, Easier Management
- grafana dashboard 지원

#### Glossary

- Keyspace
  - 논리적인 (sharded) db
  - 전체 샤드(shard의 집합)를 포함하는 논리적인 하나의 분산 DB
- Shard
  - DB(keysapce)의 데이터를 수평적으로 나눈 단위
  - 물리적으로 독립적으로 저장, 관리되는 단위
- Primary Vindex, Vindex
  - 샤드를 나누고 찾는 키
  - Sharding Key
- Keyspace ID
  - 각각의 레코드는 샤딩키에 따라서 Keyspace ID를 가지게 되고 특정 샤드에 저장됨
  - ID는 사용자 테이블에 저장되지 않고 런타임 시 계산됨

#### Architecture

[![image01](https://github.com/leeplay/study/blob/master/image/vitess-image01.png?raw=true)]

- Replication Cluster
  - Primary
  - Replica Node
- VTTablet
  - daemon이자 sidecar (mysqld + DB)
  - kube에서는 하나의 pod으로 동작
- vtgate
  - 프록시 서버
  - mysql protocol 지원
  - 전체 데이터베이스를 싱글뷰로 제공함
  - vttablet으로 쿼리를 릴레이 해줌
  - vtgate 끼리는 stateless 해 scale out이 가능

#### Scalability Philosophy

- 큰 소수의 샤드 vs 작은 다수의 샤드
- vitess에서는 작은 다수의 샤드를 권장
- 장애 시 재구축을 권장
- 샤드당 250g 권고
  - 하나의 노드의 최대 15분 이내에 re-provision 하기 위한 기준치
- Distributed Durability
  - 3 replicas (1 Primary + 2 Replicas) 추천
  - semi-sync 사용해 복제 지연을 최소화
  - 크래시 발생 시 DB를 복구하지 않고 새로운 primary에서 데이터를 가져와 복구함
- No multi-primary
  - 하나의 Primary와 여러 개의 Replicas
- Orchestartor for HA 지원

#### Consistency Model
- ACID
- READ COMMITTED 레벨로 동작
  - cross-shard query
  - 참고로 mysql은 REPEATABLE READ
- 싱글 샤드 수준의 업데이트가 발생할 있도록 데이터모델링 권고

#### 사내 지원

- 기본 1 unsharded keyspace
- 3 replicas 구성
- 샤드당 1개의 백업
- 필요시 Keyspaces, Shards 추가 가능
- Ceph RBD or local/ephemeral disk
- 개발은 자유사용, 리얼은 협의 후 제공


### Concepts

#### Vertical Sharding

- 한 DB에 모든 테이블을 포함하고 있지만 일부 테이블을 별개의 DB로 구성하고 싶을 때

- MoveTable 도구를 통해서 온라인으로 이동 가능
- image02

#### Horizontal Sharding

- Shard를 나눠서 scale out
- Reshard 통해서 온라인으로 가능
- image03

#### Keyspace, Keyspace ID

- Keyspace
  - 논리적인 하나의 데이터베이스
  - 샤드의 집합
  - 각 샤드의 keyspace id의 range로 구성됨

- Keyspace ID (sharding key)
  - 각각의 row는 하나의 id를 가짐
  - 64bit unsigned int
  - 해당 row가 속하는 샤드가 정해짐
  - 특정 record가 어느 샤드에 존재하는지 계산 가능
  - application에서 신경쓰지 않아도 됨

#### Shard

- 주어진 쿼리를 파싱하고 해당 샤드를 런타임 때 식별
- 각 shard 키값을 keyspace id로 변환하기 위한 해시펑션이 필요함
  - build-in hash function 제공
  - 일반적으로 xxhash 사용, 타입무관, 성능우수
  - equi range 샤딩 디폴트 제공
  - 커스텀 샤딩룰도 제공 가능
- scatter/gather 쿼리지원(모든 샤드 방문)
- cross-shard 지원

- VSchema
  - 하나의 논리적인 DB를 어떻게 나눌것(샤드)인지 표현
  - image04

- Vindex
  - record의 keyspace id를 찾아내는 방법
  - 사용자 쿼리를 라우팅을 하기위함
  - Primary Vindex, lookup vindex -> cross shard를 위해 사용
    - Primary Vindex
      - shard key
      - 어떻게 나누어 저장하고 어떻게 쿼리할 것인가
      - 분산 모델링에서 가장 중요한 포인트
      - 한번 정의하면 변경불가
      - 
  - mysql index와 무관함
  - image05

#### Query Planning
- 주어진 사용자쿼리와 VSchema 정보를 활용해 액세스 플랜을 생성함

#### Connectors
- MySQL 커넥터사용
- image06

#### SQL 호환성
- 샤드미적용시 그대로 사용
- 지원불가 쿼리 확인필요 
- vtexplain 도구로 수행쿼리 확인 가능

#### Perfomance
- throughput 우수, latency는 감소


## 데이터 모델링

### Sharding 전략

- shared keyspace 생성
- shard할 테이블에 대한 모델링 (VSchema 정의가 필요)

#### Primary Vindex 선정

- Cardinality
- 균질한 분포 (양, 시간 측면)
- 쿼리 최적화
- 데이터 모델
- 분산쿼리/트랜잭션
- PK or FK 선택을 해야하는데 참조관계일 경우는 FK로 해야 한 샤드에 다 저장됨
- image07

#### Shard Function

- 일반적인 경우는 xxhash 추천
- Binary나 이미 hash된 경우는 다른 전략 사용 가능

#### VSchema 관리방법

- vtctlclient의 GetVSchema command로 현재 VSchema 정의를 json으로 받아서 이를 편집

#### Lookup Vindex

- cross-shard index
- Primary Vindex로 조회불가 시 사용, RDB의 Secondary Index와 비슷
- pk가 아닌키로 조회 시 고민

#### Unique Constraint

- 샤드키가 UC에 포함된 경우
- 테이블의 Shard Key가 UK에 포함된 경우
  - 싱글 샤드 수준에서의 uniqueness가 전체 수준에서의 uniqueness 보장됨
  - 추가 조치 필요없음
- 그 외 경우
  - MySQL 수준에서 UC는 싱글샤드 수준에서만 보장됨
  - consistent lookup unique 타입의 Lookup Vindex 추가 필요

#### AUTO_INCREMENT

- Vitess의 Sequence 사용
- MySQL의 AUTO_INCREMENT 대체
- 응용쿼리 변경하지 않아도 됨
- Unsharded 테이블에 생성 후 shared에서 사용

#### UUID

- 응용에서 UUID를 생성
- 입력성능을 위해서는 단순한 UUID가 아닌 Ordered UUID가 필요
  - 타임스탬프를 앞에 위치시켜서 순서가 유지되도록 생성
  - mysql 인덱스에서 random insert가 아닌 append를 유도하기 위함

#### 복제 기반 방식

- 샤딩을 적용하면 스케일아웃 가능하지만 일부 쿼리 표현/성능이 제약됨
- 위와같은 문제해결을 위해 복제 기반의 VReplication 방식 지원
- Reference Table (global table)
  - 트랜잭션 테이블이 참조하는 테이블과의 join
  - 둘 다 shard 하려니 shard key가 서로 달라서 같은 shard에. 위치시키지 못 함, 이런 경우 cross shard join으로 처리됨
  - 성능을 위해서 cross shard join이 아닌 local join 유도 필요
  - 참조하는 코드성격의 테이블을 모든 shard로 복제해 local join으로 변경
  - source의 변경을 VReplication이 자동으로 복제
  - 복제기반이므로 복제 지연으로 인한 stale read 가능

- Materialized View
  - Where predicate에 원하는 조건에 맞는 복제도 가능
  - 특정칼럼과 집계함수에서도 사용 가능
  - Target Table정의와 이에 대한 VSchema 정의
  - Source Expression 필요에 따라서 정의


## 의문

- 개인정보를 보유한 테이블도 vitess 사용이 가능한가
- 현재 담당 DBA에게 받는 지원과 동일한가 ? 부족하다면 뭐가 빠지는가 ?
- 현재 저장서비스에서 가장 많이 사용하는 쿼리는 무엇인가 ? 샤딩 적용시 적절히 분산되는가 ?
- 샤딩키 전략은 ? cross shard join이 안생겨야함
- 권한관리는 ?
- 사용자 데이터는 어디까지 증가할까 ?
- 리샤드 발생 시 ?
