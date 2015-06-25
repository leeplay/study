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
	  - BindPodsRateLimiter util.RateLimiter
	  - Recorder record.EventRecorder
      
- func 
  - New(c *Config) *Scheduler


https://github.com/GoogleCloudPlatform/kubernetes/tree/52db576617a5d0c10d9ea30ab031f8f4609ddaf4/contrib/mesos/pkg/scheduler


you might need to change code in contrib/mesos/pkg/scheduler



