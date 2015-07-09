Overview
========

- src/plugin/pkg/scheduler 패키지에 위치함 

- 아래의 패키지로 구성되어 있음
  - algorithm 
  - algorithmprovider
  - api
  - factory
  - metrics 

- 아래의 go 파일로 구성되어 있음
  - generic_scheduler.go
  - modeler.go
  - scheduler.go
  
package Scheduler 
=================

### scheduler.go 

- interface
  - Binder
    - Bind(binding *api.Binding) error
    
  - SystemModeler
    - AssumePod(pod *api.Pod)
    - ForgetPod(pod *api.Pod)
    - ForgetPodByKey(key string)
    - LockedAction(f func())
  
- struct
  - Scheduler
    - config    Config
    - (s *Scheduler) Run()
  	  - 고루틴으로 스케쥬링과 와칭을 시작
  	
  - (s *Scheduler) scheduleOne()
  	- pod을 가져옴, 가져오기 전까지 대기 
  		- 가져온 pod을 RateLimiter가 적합하지 않으면 RateLimiter가 이용가능 상태가 될 때까지 대기 
  	- 현재 시간을 가져옴
  	- 스케쥴러의 MinionLister 의해 dest를 구함
  	- 스케쥴러에 의해 pod이 node로 바인딩됨에
  	- 스케쥴러 Binder에 바인딩된 정보를 기록
  	- 스케쥴러 Recorder에 바인딩 이벤트 기록 
  	- 스케쥴러 Modeler에 생성된 pod을 인지시킴
      
  - Config
    - Modeler      SystemModeler
    - MinionLister algorithm.MinionLister
    - Algorithm    algorithm.ScheduleAlgorithm
    - Binder       Binder
    - BindPodsRateLimiter util.RateLimiter  /pod을 생성할 수 있는 제한률?
    - Recorder record.EventRecorder
    - NextPod func() *api.Pod           // 함수형 변수, 다음 pod을 얻을 때까지 블록 상태 
    - Error func(*api.Pod, error)
    - Recorder record.EventRecorder
    - StopEverything chan struct{}
      
- func 
  - New(c *Config) *Scheduler()
  	- 주어진 config 정보로 스케쥴러 생성과 메트릭스에 등록함
  	

### modeler.go

- interface
  - ExtendedPodLister                                 // simpleModeler에서 pod들을 리스팅 하는 것외에 기존 pod체크
    - algorithm.PodLister
    - Exists(pod *api.Pod) (bool, error)

- struct 
  - actionLocker                                     // fake와 sympleModler에서 사용, lockedAction과 병행해서 사용
    - sync.Mutex
    - LockedAction(do func())
  
  - FakeModeler                                      // SystemModeler 인터페이스를 구현 
    - AssumePodFunc      func(pod *api.Pod)
    - ForgetPodFunc      func(pod *api.Pod)
    - ForgetPodByKeyFunc func(key string)
    - actionLocker                             // actionLocker 상속 
    - AssumePod(pod *api.Pod)
    - ForgetPod(pod *api.Pod)
    - ForgetPodByKey(key string) 
  
  - SimpleModeler                                    // SystemModeler 인터페이스를 timed pod cache로 구현 
    - queuedPods    ExtendedPodLister      
      - 아직 스케쥴링이 안된 pod을 리턴
    - scheduledPods ExtendedPodLister   
      - 스케쥴링이 된 pod을 리턴 
    - assumedPods *cache.StoreToPodLister   
    - actionLocker  // actionLocker 상속
    - AssumePod(pod *api.Pod)
    - ForgetPod(pod *api.Pod)
    - ForgetPodByKey(key string)
    - listPods(selector labels.Selector) (pods []*api.Pod, err error)
      - assumedPods.List(selector)호출해  assumed pod list을 가져옴 
      - 반복문시작
        - queuedPods에 가져온 aussmed pod이 존재하면 assumedPod.Store에서 삭제 후 반복문 다시 시작
        - scheduledPods에 가져온 assumed pod이 존재하면 assumedPod.Store에서 삭제 후 반복문 다시 시작
      - scheduledPods.List(selector)호출해 scheduledPods pod list을 가져옴 
      - s.assumedPods.List(selector)호출해 assumed pod list을 가져옴 
        - assumed 가 0이면 scheduled를 리턴
        - 0이 아니면 scheduled + assumed를 리턴 
    - PodLister() algorithm.PodLister
      - simpleModelerPods을 생성, 스케쥴링이 된 pods을 리턴
      
  - simpleModelerPods        // simpleModelerPods은 simpleModler를 PodLister로 하기위한 아답터 입니다. 
  	- simpleModeler *SimpleModeler
  	- List(selector labels.Selector) (pods []*api.Pod, err error) 
  	  - assumed 이거나 알려진 pods을 list 형태로 리턴

- func
  - NewSimpleModeler(queuedPods, scheduledPods ExtendedPodLister) *SimpleModeler
  - podNames(pods []*api.Pod) []string
    - pods의 pod들의 네임스페이스, 네임, UID 정보를 가져옴

### generic_scheduler.go

  - variable 
    - FailedPredicateMap map[string]util.StringSet
    - var ErrNoNodesAvailable = fmt.Errorf("no nodes available to schedule pods")
  
  - func 
    - findNodesThatFit(pod *api.Pod, podLister algorithm.PodLister, predicateFuncs map[string]algorithm.FitPredicate, nodes api.NodeList) (api.NodeList, FailedPredicateMap, error)
    	- MapPodsToMachines()를 호출해 스케쥴은 되어있는데 실행되지 않는 pod을 구함 
    	- 주어진 노드만큼 순회
    		- 주어진 predicateFuncs만큼 순회
    			- 실행되지 않은 pod이 기존 노드에 들어가는 게 적합한지 판단
    				- 적합하지 않으면 failedPredicateMap에 추가 후 노드 순회 반복문으로 빠짐 
    				- 적합하면 []api.Noden{} filtered에 추가 후 노드 순회 반복문으로 
    	- filtered, fialedPredicateMap을 리턴 
    	
    - prioritizeNodes(pod *api.Pod, podLister algorithm.PodLister, priorityConfigs []algorithm.PriorityConfig, minionLister algorithm.MinionLister) (algorithm.HostPriorityList, error)
    	- priorityConfigs가 0이면 EqualPriority를 호출해 HostPriorityList를 전부 동일하게 1로 리턴, 숫자가 높을 수록 좋음 
    	- combinedScores 생성
    	- priorityConfigs 만큼 순회
    		- priorityConfig.Weight를 구함 
    		- priorityConfig.Function 로 prioritizedList(특정 호스트로 가기 위한 스케쥴링의 우선순위, 낮을 수록 좋음)를 구함
    		- prioritizedList 만큼 순회
    			- hostEntry.Score * weight 해서 combinedScores에 넣음 
    	- combinedScores와 result를 합친 HostPriorityList 를 리턴 
    	
    - getBestHosts(list algorithm.HostPriorityList) []string
    	- list 0 번째가 항상 최우선인듯, 그 녀석과 동급인 hostEntry 정보를 전부 가져옴  
    	
    - EqualPriority(_ *api.Pod, podLister algorithm.PodLister, minionLister algorithm.MinionLister) (algorithm.HostPriorityList, error)
    	- minion의 우선순위를 1로 동일하게 만듬
    
    - NewGenericScheduler(predicates map[string]algorithm.FitPredicate, prioritizers []algorithm.PriorityConfig, pods algorithm.PodLister, random *rand.Rand) algorithm.ScheduleAlgorithm
    	- genericscheduler 생성

  - struct
    - FitError 
      - Pod *api.Pod
      - FailedPredicates FailedPredicateMap
      - Error() string
  
   - genericScheduler
      - predicates   map[string]algorithm.FitPredicate
      - prioritizers []algorithm.PriorityConfig
      - pods         algorithm.PodLister
      - random       *rand.Rand
      - randomLock   sync.Mutex
      - Schedule(pod *api.Pod, minionLister algorithm.MinionLister) (string, error)
      	- findNodesThatFit을 호출해 filteredNodes, failedPredicateMap 를 구함 
      	- prioritizeNodes를 호출해 priorityList를 구함
      	- selectHost를 호출해 host를 구함

      - selectHost(priorityList algorithm.HostPriorityList) (string, error)
      	- scheduler를 위한 minion의 list를 가져옴
      	- 역순으로 정렬한다
      	- 가장 적합한 호스트를 구한다. 
      	- 적합한 호스트 중 한 곳을 랜덤으로 가져온다. 
 
 
package metrics 
=================

### metrics.go

- variable
	- schedulerSubsystem 
	- E2eSchedulingLatency    
	- SchedulingAlgorithmLatency
	- BindingLatency
		- Scheduler의 scheduleOne()에서 latency 시간 정보를 기록

- func 
	- Register()
		- sheduler 생성 시 최초 한번만 실행 
	- SinceInMicroseconds()
		- 시작시간부터 현재시간까지 나노초로 구함


package factory 
=================

- factory.go
	- variable
		- BindPodsQps   = 15
		- BindPodsBurst = 20
	
	- struct
		- ConfigFactory
			- Client *client.Client
			- PodQueue *cache.FIFO 
				- 스케쥴링이 필요한 큐 
			- ScheduledPodLister *cache.StoreToPodLister
				- 스케쥴 설정된 pods의 리스트 정보를 가짐
			- PodLister algorithm.PodLister
				- 스케쥴 설정된 pods와 스케쥴 되어야 할 pods로 추정되는 리스트
			- NodeLister *cache.StoreToNodeLister
				- 모든 minion 리스트 
			- ServiceLister *cache.StoreToServiceLister
				- 모든 service 리스트 
			- StopEverything chan struct{}
			- BindPodsRateLimiter util.RateLimiter
			- scheduledPodPopulator *framework.Controller
			- modeler               scheduler.SystemModeler
			
			- Create() (*scheduler.Config, error)
				- 디폴트 알고리즘프로바이더로 스케쥴러 생성 
			
			- CreateFromProvider(providerName string) (*scheduler.Config, error)
				- GetAlgorithmProvider로 알고리즘을 구함
				- CreateFromKeys로 등록된 fit predicate key와 priority key로 구성된 set으로부터 스케쥴러 생성 
				- 알고리즘프로바이더로 스케쥴러 생성 
			
			- CreateFromConfig(policy schedulerapi.Policy) (*scheduler.Config, error)
				- config파일로부터 스케쥴러 생성 
				
			
			- CreateFromKeys(predicateKeys, priorityKeys util.StringSet) (*scheduler.Config, error)
			- createUnassignedPodLW() *cache.ListWatch
			- createAssignedPodLW() *cache.ListWatch
			- createMinionLW() *cache.ListWatch
			- reateServiceLW() *cache.ListWatch
			- makeDefaultErrorFunc(backoff *podBackoff, podQueue *cache.FIFO) func(pod *api.Pod, err error)
				- 클러스터에 등록된 노드가 없다 
				- 스케쥴링 에러다 다시 시도해라 
				- gc로 1차 정리 
				- 
			
			
		- nodeEnumerator
			- *api.NodeList
		
			- Len() int
				- node list의 아이템의 갯수 리턴
		
			- Get(index int) interface{}
				- 주어진 인덱스의 아이템을 리턴
		
		- type binder struct
			- *client.Client
			- Bind(binding *api.Binding) error
				- rpc  호출인거 같은데 현재 구현안되어 있음
		
		- realClock 
			- Now() time.Time
				- 현재시간 리턴 
		
		- backoffEntry 
			- backoff    time.Duration
				- 두 개의 인스턴스 사이의 흐른시간 ?
			- lastUpdate time.Time
		
		- podBackoff
			- perPodBackoff   map[string]*backoffEntry
			- lock            sync.Mutex
			- clock           clock
			- defaultDuration time.Duration
			- maxDuration     time.Duration
			- getEntry(podID string) *backoffEntry
				-  주어진 podId에 해당하는 *backoffEntry의 lastUpdate 시간을 현재시각으로 설정하고  리턴 
	
			- getBackoff(podID string) time.Duration
				- getEntry
				- backoff 시간을 2로 곱합 
				- backoff 시간이 maxDuratio 시간보다 크면 backoff 시간에 maxDuration 시간을 입력
				- backoff 리턴
			
			- wait(podID string)
				- getBackoff 시간만큼 sleep 
				
			- gc()
				- perPodBackoff 만큼 순회
					- perPodBackoff에서 가져온 엔트리의 마지막 업데이트 시간과 현재 시간을 비교
						- 비교 시간이 맥스지연시간보다 크면 perPodBackoff에서 삭제 
	
	- interface
		- clock 
			- Now() time.Time
	
	- func
		- NewConfigFactory(client *client.Client) *ConfigFactory
		- parseSelectorOrDie(s string) fields.Selector
		
}
		
		


- plugins.go 

package algorithmprovider 
=========================

### plugings.go 
	- 아직 빈 파일 
	- _ "github.com/GoogleCloudPlatform/kubernetes/plugin/pkg/scheduler/algorithmprovider/defaults"
		- 이런 참조로 defaults의 init을 먼저 호출해 factory에 등록한다. 아마 향후에 다른 알고리즘이 등록이 될 거 같다. 

package algorithmprovider/defaults
==================================

- func 
	- init()
		- factory.RegisterAlgorithmProvider(factory.DefaultProvider, defaultPredicates(), defaultPriorities())
		- factory.RegisterPriorityFunction("EqualPriority", scheduler.EqualPriority, 1)
