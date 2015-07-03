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
    - (s *Scheduler) scheduleOne()
      
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



