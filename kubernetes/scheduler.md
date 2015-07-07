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
