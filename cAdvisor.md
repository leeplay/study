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


메인 함수   
==========

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

	// Inmemory-backendStorage 생성 (eg. influxdb)
	memoryStorage, err := NewMemoryStorage(*argDbDriver)
	if err != nil {
		glog.Fatalf("Failed to connect to database: %s", err)
	}

	// block-device(eg. device, device size, device count), network-device(networkdevice, networkaddress, MTU, networkspeed), Cache, SystemUUID
	sysFs, err := sysfs.NewRealSysFs()
	if err != nil {
		glog.Fatalf("Failed to create a system interface: %s", err)
	}

	// memory, storage 관리 manager 생성
	containerManager, err := manager.New(memoryStorage, sysFs)
	if err != nil {
		glog.Fatalf("Failed to create a Container Manager: %s", err)
	}

	mux := http.DefaultServeMux

...
}
```

### manager 생성 단계  

```
type manager struct {
	containers             map[namespacedContainerName]*containerData  // 네임스페이스와 컨테이너명 
	containersLock         sync.RWMutex
	memoryStorage          *memory.InMemoryStorage  // influxdb 정보
	fsInfo                 fs.FsInfo        // docker rootdir을 최상위로 한 파일시스템 stat 정보
	machineInfo            info.MachineInfo  // /proc/cpuinfo, meminfo 등등 
	versionInfo            info.VersionInfo  // kernel, docker, os, cadvisor 버전
	quitChannels           []chan error      // watcher 등록
	cadvisorContainer      string              // docker가 사용 중인 cgroup 경로   
	dockerContainersRegexp *regexp.Regexp
	loadReader             cpuload.CpuLoadReader  // kernel module과 user space process간 통신을 위한  netlink 생성, cpu 사용량을 커널에게 받음  
	eventHandler           events.EventManager   // 이벤트 핸들러 등록 (eg. 컨테이너 생명 주기 이벤트)
	startupTime            time.Time           // mangager 생성 시간
}
```

- machineInfo

```
kyu@kyu-HP-EliteBook-2570p:/proc$ cat cpuinfo
processor	: 0
vendor_id	: GenuineIntel
cpu family	: 6
model		: 58
model name	: Intel(R) Core(TM) i7-3520M CPU @ 2.90GHz
stepping	: 9
microcode	: 0x16
cpu MHz		: 2901.000
cache size	: 4096 KB
physical id	: 0
siblings	: 4
core id		: 0
cpu cores	: 2
apicid		: 0
initial apicid	: 0
fpu		: yes
fpu_exception	: yes
cpuid level	: 13
wp		: yes
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx rdtscp lm constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc aperfmperf eagerfpu pni pclmulqdq dtes64 monitor ds_cpl vmx smx est tm2 ssse3 cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm ida arat epb xsaveopt pln pts dtherm tpr_shadow vnmi flexpriority ept vpid fsgsbase smep erms
bogomips	: 5786.96
clflush size	: 64
cache_alignment	: 64
address sizes	: 36 bits physical, 48 bits virtual
power management:
```

```
kyu@kyu-HP-EliteBook-2570p:/proc$ cat meminfo
MemTotal:        8037044 kB
MemFree:         2873012 kB
Buffers:          482008 kB
Cached:          3374120 kB
SwapCached:            0 kB
Active:          1826412 kB
Inactive:        2608668 kB
Active(anon):     589480 kB
Inactive(anon):   180140 kB
Active(file):    1236932 kB
Inactive(file):  2428528 kB
Unevictable:       37032 kB
Mlocked:           37032 kB
SwapTotal:       8247292 kB
SwapFree:        8247292 kB
Dirty:                 0 kB
Writeback:             0 kB
AnonPages:        616084 kB
Mapped:           180684 kB
Shmem:            181824 kB
Slab:             567196 kB
SReclaimable:     529320 kB
SUnreclaim:        37876 kB
KernelStack:        4432 kB
PageTables:        26524 kB
NFS_Unstable:          0 kB
Bounce:                0 kB
WritebackTmp:          0 kB
CommitLimit:    12265812 kB
Committed_AS:    3619080 kB
VmallocTotal:   34359738367 kB
VmallocUsed:      554616 kB
VmallocChunk:   34359161208 kB
HardwareCorrupted:     0 kB
AnonHugePages:    303104 kB
HugePages_Total:       0
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
DirectMap4k:      120772 kB
DirectMap2M:     8128512 kB
```

```
machineInfo := &info.MachineInfo{
		NumCores:        ,
		CpuFrequency:   clockSpeed,
		MemoryCapacity: memoryCapacity,
		DiskMap:        diskMap,
		NetworkDevices: netDevices,
		Topology:       topology,
		MachineID:      getInfoFromFiles(*machineIdFilePath),
		SystemUUID:     systemUUID,
		BootID:         getInfoFromFiles(*bootIdFilePath),
	}
```

- docker container factory 생성
	docker daemon과 통신하는 클라이언트 생성 (eg. aufs, libcontainer를 사용 중인지 확인한다. docker가 정상으로 실행 중인지 확인)

- raw driver factory 생성

```
	var supportedSubsystems map[string]struct{} = map[string]struct{}{
	"cpu":     {},
	"cpuacct": {},
	"memory":  {},
	"cpuset":  {},
	"blkio":   {},
}
```	

### manager 실행 단계  

- syscall용 netlink 생성
- cpu load 

```
func newConnection() (*Connection, error) {

	fd, err := syscall.Socket(syscall.AF_NETLINK, syscall.SOCK_DGRAM, syscall.NETLINK_GENERIC)
	if err != nil {
		return nil, err
	}

	conn := new(Connection)
	conn.fd = fd
	conn.seq = 0
	conn.pid = uint32(os.Getpid())
	conn.addr.Family = syscall.AF_NETLINK
	conn.rbuf = bufio.NewReader(conn)
	err = syscall.Bind(fd, &conn.addr)
	if err != nil {
		syscall.Close(fd)
		return nil, err
	}
	return conn, err
}
```

- Watch for OOMs 

- "/" 컨테이너 생성 (diff 계산을 위한 루트 설정)

- 모든 컨테이너 로딩

- 새로운 컨테이너 watch용 스레드 생성 


컨테이너 생명 주기
==================

- 100ms(default) 기준으로 "/"에 변동된 container 가 있는지 확인
- 주어진 namespace 기반으로 현재 실행 중인 컨테이너를 찾음 (eg. docker, user)

```
func (m *manager) getContainersDiff(containerName string) (added []info.ContainerReference, removed []info.ContainerReference, err error) {
```

![/](https://github.com/leeplay/study/blob/master/etc/cadvisor_root.png?raw=true)


- 





