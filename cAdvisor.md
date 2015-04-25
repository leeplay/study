cAdvisor
========

cAdvisor(Container Advisor)는 lmctfy와 docker 컨테이너를 모니터링 하기 위해 구글에서 만들었습니다. cAdvisor를 통해 실행 중인 컨테이너의 자세한 리소스 사용량과 다양한 성능 수치 자료를 얻을 수 있습니다. cAdvisor는 데이터 확인을 위한 간단한 UI와 data를 얻기 위한 API 그리고 InfluxDB에 데이터 저장을 할 수 있습니다.

만약 호스트에서 실행 중인 모든 컨테이너의 모니터링을 원한다면 cAdvisor 이미지를 동일한 호스트에서 실행시키면 됩니다. Docker version 0.11 위에서 실행할 수 있습니다. 

```
sudo docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:rw \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  google/cadvisor:latest
```

실행 후 아래 url에서 확인 가능합니다. 
http://localhost:8080 

모니터링 확인용으로 다른 샘플 컨테이너 이미지를 제공합니다.
docker run -d borja/unixbench

호스트 리소스의 오버뷰를 제공합니다. /docker를 클릭하면 새로운 창으로 컨테이너 개별 리스트를 확인할 수 있습니다. 
json object 형태의 컨테이너의 모든 정보를 얻을 수 있는 api도 제공합니다.

cAdvisor 수집 데이터 
====================

- cadvisor.go 

``` go
func main() {
	defer glog.Flush()
	flag.Parse()

	if *versionFlag {
		fmt.Printf("cAdvisor version %s\n", version.VERSION)
		os.Exit(0)
	}

	setMaxProcs()

	memoryStorage, err := NewMemoryStorage(*argDbDriver)
	if err != nil {
		glog.Fatalf("Failed to connect to database: %s", err)
	}

	sysFs, err := sysfs.NewRealSysFs()
	if err != nil {
		glog.Fatalf("Failed to create a system interface: %s", err)
	}

	containerManager, err := manager.New(memoryStorage, sysFs)
	if err != nil {
		glog.Fatalf("Failed to create a Container Manager: %s", err)
	}

	mux := http.DefaultServeMux

	// Register all HTTP handlers.
	err = cadvisorHttp.RegisterHandlers(mux, containerManager, *httpAuthFile, *httpAuthRealm, *httpDigestFile, *httpDigestRealm, *prometheusEndpoint)
	if err != nil {
		glog.Fatalf("Failed to register HTTP handlers: %v", err)
	}

	// Start the manager.
	if err := containerManager.Start(); err != nil {
		glog.Fatalf("Failed to start container manager: %v", err)
	}

	// Install signal handler.
	installSignalHandler(containerManager)

	glog.Infof("Starting cAdvisor version: %q on port %d", version.VERSION, *argPort)

	addr := fmt.Sprintf("%s:%d", *argIp, *argPort)
	glog.Fatal(http.ListenAndServe(addr, nil))
}
```

- manager.go 

```
type manager struct {
	containers             map[namespacedContainerName]*containerData
	containersLock         sync.RWMutex
	memoryStorage          *memory.InMemoryStorage
	fsInfo                 fs.FsInfo
	machineInfo            info.MachineInfo
	versionInfo            info.VersionInfo
	quitChannels           []chan error
	cadvisorContainer      string
	dockerContainersRegexp *regexp.Regexp
	loadReader             cpuload.CpuLoadReader
	eventHandler           events.EventManager
	startupTime            time.Time
}
```

### SubContainers
  - manager.go
  - 

cAdvisor 데이터 수집방법
========================




