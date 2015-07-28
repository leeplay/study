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
	- 가장 적합한 노드들을 필터링함
		- 스케쥴될 pod, 모든 pod의 리스트, 모든 minions 로 시작 
		- terminated상태가 아닌 pod들을 구함, 즉 현재 실행 중인 pods 
		- node list 만큼 반복
			- predicateFuncs 만큼 반복 
				- predicateFunc로 node에 pod을 배포하기 적합한지 확인 
		- fit 한 nodes만 리턴
	- node list의 minion list를 구함
	- minion list 중 가장 우선순위가 높음 node를 찾음
		- priority config에 아무런 내용이 없다면 전부 동일한 priority를 받음 
		- priority config 만큼 반복 
			- priorityFunc로 minion list의 score를 구함
			- minion list 만큼 반복
				- minion의 score와 priorityconfig의 weight을 곱함 
	- 가장 우선순위가 높은 노드들 중 랜덤으로 하나의 호스트를 선택
	
- 바인딩 메타정보 생성 
- 바인딩 시도
	- 바인딩될 pod을 RPC를 호출하여 바인딩
	- pod이 스케쥴링이 되었다는 정보를 SimpleModeler에 저장



주요 알고리즘 : predicate  
========================

- 배포할 pod의 가장 적합한 node list를 찾는 알고리즘

### node 리소스 확인 알고리즘  

```
- pod의 요청 리소스를 구함
- 배포지정된 node의 상태정보(현재 상태 정보는 cpu, memory, storage)를 구함
- 배포될 pod의 request정보(cpu, memory)가 0 이면
	- node가 가질 수 있는 pod의 갯수가 existing pod의 갯수보다 크면 true 작으면 false 리턴
- 기존 pod list를 새로운 pod list로 복사 
- 새로운 pod list에 배포할 pod 추가
- node의 capacity를 초과하는지, 모든 pod list의 fit 여부를 확인 
	- node의  토탈 cpu, memory를 구함 
	- existing pod의 모든 pod을 체크 <- 굳이 왜 ???, 새롭게 추가된 것만 하지지
		- 전체 리소스양과 pod별 요청 리소스양을 체크해서 전체 리소스양을 초과하는지 확인 
		- pod을 fit한 것과 doesnt'fit한 것과 나눔
- 부적합한 fit이 발견되면 fasle 를, 전부 적합하면 true를 리턴 
```

### node 존재 확인 알고리즘  

```
- pod에 지정된 node의 이름이 존재하는지 확인
```

### port 배포가능 확인 알고리즘 

```
- 기존 pod list의 port를 구함 
- 배포될 pod의 port를 구함 
- 비교후 true, false 리턴함
```

### Disk (GCE, ISCSI, AWS, Git, Secret, NFS, Gluster, RBD) 충돌 확인 알고리즘 

```
- 배포될 Pod의 Volumes 정보를 구함
	- Volumes 갯수만큼 반복
		- 기존 pos list 만큼 반복
			- volume이 충돌나는지 확인(GCE, AWS만 구현되어 있음)   <- 코드 기여여부가 보이네요 
				- pod이 가진 volume과 기존 pod들간의 volume 명칭이 일치하는지 확인
				- 추가로 volume 벤더별 상태값들을 몇개 더 체크함 
		- 하나라도 맞지 않으면 도중에 false로 빠져나옴 
	- 이상없으면 true 리턴
```

### Selector 체크 알고리즘 

```
// 아래처럼 node를 선택하는 selector를 가짐

Spec: api.PodSpec{
	NodeSelector: map[string]string{
	"foo": "bar",
	"baz": "blah",
	},
}

labels: map[string]string{
	"foo": "bar",
}

```

```
- 노드에 존재하는 minion list를 구함 
- 배포할 pod의 selector가 없다면 true 리턴 
- 배포할 pod의 selector와 node의 label이 일치하는지 확인 
```

### LabelPresence 체크 알고리즘

```
// 아래처럼 label의 존재와 presence의 값이 일치하는지 확인하는 알고리즘 


{
	labels:   []string{"foo", "bar"},
	presence: true,
	fits:     true,
	test:     "all labels match, presence true",
}

// 처음엔 pod을 사용하는 줄 알았는데 전혀 사용하지 않음, 불필요한 인자가 있을 순 있지만 이건 혼란을 주기 때문에... -> 코드 개선 ? 

func (n *NodeLabelChecker) CheckNodeLabelPresence(pod *api.Pod, existingPods []*api.Pod, node string) (bool, error) {
	var exists bool
	minion, err := n.info.GetNodeInfo(node)
	if err != nil {
		return false, err
	}
	minionLabels := labels.Set(minion.Labels)
	for _, label := range n.labels {
		exists = minionLabels.Has(label)
		if (exists && !n.presence) || (!exists && n.presence) {
			return false, nil
		}
	}
	return true, nil
}
```

```
- 노드에 존재하는 minion list를 구함 
- minion list 들의 label list 정보를 구함
	- label list만큼 반복
		- minion이 label을 가지고 있는지 확인
			- 가지고 있다면 node의 presence정보와 일치하는지 확인 
```

### Service 유사성 체크 알고리즘 

```
- 배포할 pod의 nodeselector 정보를 구함 
- nodeselector의 label  정보를 구함
- Service List의 label 만큼 반복
	- pod의 nodeselector 정보가 label의 정보와 일치하면 affinityLabels에 저장 -> 유사하다고 판단된 것 
	- pod의 nodeselector가 하나라도 맞지않는게 있다면 labelExist에 false를 저장  
- labelExist이 false일 경우 모든 서비스 리스트를 가져와서 비교함 
	- label 정보가 일치하는 service list를 구함
	- service list만큼 반복 
		- 동일한 service이기 때문에 항상 0번째에서 selector를 구함 
		- 해당 service에서 pod list를 구함 
		- service의 pod list만큼 반복
			- service의 pod과 배포할 pod의 네임스페이스가 같다면 nsServicePods저장 
		- nsServicePods 가 0보다 클 경우
			- nsServicePods의 노드이름으로 ServiceAffinity의 노드 정보를 구함 -> 다른 minion을 구한 거임 
			- ServiceAffinity의 label 만큼 반복 -> 설정한 label만큼 반복하겠다는 얘기임
			- label과 다른 minion의 label이 같다면 affinityLabels저장함 
- 뭔소리인가 싶겠지만 여기까지 오면 affinityLabels 정보가 생성됩니다. 
- affinityLabels에서 affinitySelector를 꺼냄
- node의 minion을 구함 
- affinitySelector와 minion의 label이 일치하는지 판단
```

```
//글자로는 이해하기 힘드니 문제를 내겠습니다. 

selector := map[string]string{"foo": "bar"}
labels1 := map[string]string{
	"region": "r1",
	"zone":   "z11",
}
labels2 := map[string]string{
	"region": "r1",
	"zone":   "z12",
}
labels3 := map[string]string{
	"region": "r2",
	"zone":   "z21",
}
labels4 := map[string]string{
	"region": "r2",
	"zone":   "z22",
}

node1 := api.Node{ObjectMeta: api.ObjectMeta{Name: "machine1", Labels: labels1}}
node2 := api.Node{ObjectMeta: api.ObjectMeta{Name: "machine2", Labels: labels2}}
node3 := api.Node{ObjectMeta: api.ObjectMeta{Name: "machine3", Labels: labels3}}
node4 := api.Node{ObjectMeta: api.ObjectMeta{Name: "machine4", Labels: labels4}}
node5 := api.Node{ObjectMeta: api.ObjectMeta{Name: "machine5", Labels: labels4}}
}

//아래 스케쥴링 결과를 예상해보시오 ?
{
	pod:    new(api.Pod),
	node:   "machine1",
	labels: []string{"region"},
},

// 아래 스케쥴링 결과를 예상해보시오 ?
{
	pod:    &api.Pod{Spec: api.PodSpec{NodeSelector: map[string]string{"region": "r2"}}},
	node:   "machine1",
	labels: []string{"region"},
}

// 아래 스케쥴링 결과를 예상해보시오 ?
{
	pod:      &api.Pod{ObjectMeta: api.ObjectMeta{Labels: selector}},
	pods:     []*api.Pod{{Spec: api.PodSpec{NodeName: "machine1"}, ObjectMeta: api.ObjectMeta{Labels: selector}}},
	node:     "machine1",
	services: []api.Service{{Spec: api.ServiceSpec{Selector: selector}}},
}
	


	

// 다 맞추시면 스케쥴링 전문가가 되셨습니다. 		
정답 : true, false, true

// node가 가진 label 정보에 논리적으로 구역을 정함, region 정보의 일치성만 판단함, node정보가 틀려도 fit하다고 판단함, 아마 노드 정보는 별도의 메소드에서 체크하니 그런거 같음  
```

주요 알고리즘 : spreading 
========================

- 배포할 pod과 유사한 service list들에 우선순위를 부여하는 알고리즘

### pod을 배포할 서비스 리스트를 찾는 알고리즘 

```
- 전체 서비스 리스트를 구함 
- 배포할 pod의 정보, pod list의 정보, minion list의 정보를 구함 
	- pod과 동일한 서비스 label을 가진 pod list를 구함 
		- 구한 pod list와 동일한 네임스페이스를 가진 pod list를 다시 구함
- 전체 minion list를 구함
- 위에서 구한 동일한 네임스페이스를 가진 pod list만큼 반복
	- pod이 가진 node.name 이 동일한 만큼 카운트를 증가시킴
		- 만약 max count를 초과하면 증가시키지 않음, 그럼 여기서 동일한 서비스, 노드의 네임스페이스를 가진 pod들이 구해짐
- 모든 node list의 우선순위를 정함
	- 위에서 max count를 못 구한 node 라면 디폴트 값으로 채워짐 
	- 우선순위의 score를 정하는 수식임, maxCount-counts[minion.Name]) / float32(maxCount) 
	- 0부터 10까지인데 높을수록 우선순위가 높음
- algorithm.HostPriority{Host: minion.Name, Score: int(fScore)} 이런 식으로 모든 minion에 score가 매겨짐, 이렇게되면 label정보가 같은, 배포가 적게 된 node에 더 먼저 배포가 되게 됨 
```

```

```

### 존 확산 알고리즘

```
```

주요 알고리즘 : priority 
========================

- 부여된 우선순위들 중 가장 적합한 호스트를 찾는 알고리즘 

```
진행 중 
```



당장 코드 기여 가능할 곳 ???
========================

비동기 부사 표현 오타 4곳, 사용하지 않는 인자 3개 (PodSelectorMatches, CheckNodeLabelPresence)



### 나에게 던지는 질문

- 특별히 노드를 정하는 부분은 없는데, 디폴트 노드 선택 알고리즘은 어떤거지 ?
- 쿠베 기동시 정의되어 있는 pod을 인지하고 실행하는 것과 기동 후에 명령어를 통해서 실행하는 건 차이가 있는지 ?
- pod에 리소스 요구기준이 없을 때 할당하는 디폴트 리소스 값 ?
- pod의 실행 단위 안에 컨테이너 갯수를 설정이 되나 ?
- 쿠베 기동 시 기존 노드의 리소스 정보 파악은 어떤 식인지? 
- 쿠베 기동 후 신규 노드의 리소스 정보 파악은 어떤 식인지
- 왜 presence 를 써야만 했을까 ?

