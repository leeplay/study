Overview
========

### 서버 생성 과정 

```
func (s *SchedulerServer) Run(_ []string) error {
	
	-> Go 스레드로 헬스체크와 프로파일링용 서버 생성
	-> ConfigFactory 생성
	-> Config 생성
	-> event broadcaster 생성 후 scheduler 생성을 알림 
	-> 스케쥴러 생성 
	-> 스케쥴러 실행 
	-> 별도의 고루틴으로 왓칭과 스케쥴링 작업을 수행 
}
```
	

주요 모듈 
=========

### ConfigFactory

- Store 생성
	- PodQueue : 스케쥴링이 필요한 pod을 담은 큐
	- ScheduledPodLister : 스케쥴링이된 pod list
	- NodeLister : 모든 minion list
	- ServiceLister : 모든 service List
	- PodLister : 스케쥴링이 된 pod과 스케쥴링이 될 pod의 list

- modeler 생성 
	- simpleModeler 사용

- pod 트래픽 제어 리미터 생성 
	- NewTokenBucketRateLimiter 사용 

### Config

```
"kind" : "Policy",
"apiVersion" : "v1",
"predicates" : [
	{"name" : "TestZoneAffinity", "argument" : {"serviceAffinity" : {"labels" : ["zone"]}}},
	{"name" : "TestRequireZone", "argument" : {"labelsPresence" : {"labels" : ["zone"], "presence" : true}}},
	{"name" : "PredicateOne"},
	{"name" : "PredicateTwo"}
],
"priorities" : [
	{"name" : "RackSpread", "weight" : 3, "argument" : {"serviceAntiAffinity" : {"label" : "rack"}}},
	{"name" : "PriorityOne", "weight" : 2},
	{"name" : "PriorityTwo", "weight" : 1}		]
}
```

- kubeconfig 파일로부터 policy 생성
	- predicate 생성 
		- NewServiceAffinityPredicate 생성
		- NewNodeLabelPredicate 생성
	- priority 생성
		- NewServiceAntiAffinityPriority 생성
		- NewNodeLabelPriority 생성 

	- getFitPredicateFunctions 생성
	- getPriorityFunctionConfigs 생성 
- modeler에서 사용하는 pod remove용 고루틴 생성
- 모든 서비스 오브젝트를 watch, cache 시작

### Scheduler 

- generic-sheduler 사용 
- 고루틴으로 되어있어 계속 반복 
- pod을 가져옴 
	- Pod의 트래픽이 적정한지 확인
		- 스케쥴링이 될 pod의 트래픽을 제어
		- 별도의 고루틴에서 ticker(정해진 time duration, 0.015초)만큼 이벤트가 발생하여 token 채널에 true가 입력되고 있음, 최대 20개 입력됨 
		- token에 true인게 있으면 트래픽이 적절하다고 판단
		- token bucket 알고리즘과 비슷한 알고리즘을 사용
		- 두 개의 고루틴이 한쪽에 true를 입력하고 한쪽은 true을 가져가니 ticker 시간 내에 최대 20개까지 처리, 그러므로 누락은 없을 거 같음
		
- 스케쥴링 시도 
	- 모든 minion을 구함
	- 가장 적합한 노드들을 찾음 
		- 스케쥴될 pod, 모든 pod의 리스트, 모든 minions 로 시작 
		- Succeed와 Failed가 아닌 pod들을 구함 
		- 
	- 적합한 노드들 중 가장 우선순위가 높은 노드를 찾음
	- 가장 우선순위가 높은 노드들 중 가장 적합한 호스트를 찾음  
	
- 바인딩 메타정보 생성 
- 바인딩 시도
	- 바인딩될 pod을 RPC를 호출하여 바인딩
	- pod이 스케쥴링이 되었다는 정보를 SimpleModeler에 저장
