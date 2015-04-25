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

```
longHousekeeping := 100 * time.Millisecond
if *HousekeepingInterval/2 < longHousekeeping {
	longHousekeeping = *HousekeepingInterval / 2
}
```

- 주어진 namespace 기반으로 현재 실행 중인 컨테이너를 찾음 (eg. docker, user)

```
func (m *manager) getContainersDiff(containerName string) (added []info.ContainerReference, removed []info.ContainerReference, err error) {
```

![/](https://github.com/leeplay/study/blob/master/etc/cadvisor_root.png?raw=true)


- 새로운 컨테이너가 있다면 manager에 추가, 이 때 manager가 가지고 있는 시스템 전체 정보가 함께 넘어간다. 

```
cont, err := newContainerData(containerName, m.memoryStorage, handler, m.loadReader, logUsage)
```

- 삭제된 컨테이너가 있다면 manager에서 삭제

```
delete(m.containers, namespacedName)  // map에서 삭제 
```

컨테이너 모니터링 정보 
======================

- manager 구조체에서 시스템의 전체적인 정보를 관리했다면 container 당 개별정보 관리는 containerData 구조체이서 이뤄진다. 

```
type containerData struct {
	handler              container.ContainerHandler
	info                 containerInfo
	memoryStorage        *memory.InMemoryStorage
	lock                 sync.Mutex
	loadReader           cpuload.CpuLoadReader
	summaryReader        *summary.StatsSummary
	loadAvg              float64 // smoothed load average seen so far.
	housekeepingInterval time.Duration
	lastUpdatedTime      time.Time
	lastErrorTime        time.Time

	// Whether to log the usage of this container when it is updated.
	logUsage bool

	// Tells the container to stop.
	stop chan bool
}
```

### Isolation 

![isolation](https://github.com/leeplay/study/blob/master/etc/cadvisor_isolation.png?raw=true)

- Shares
- Allowed Cores 
- Limit
- Swap Limit 

```
func libcontainerConfigToContainerSpec(config *libcontainerConfigs.Config, mi *info.MachineInfo) info.ContainerSpec {
	var spec info.ContainerSpec
	spec.HasMemory = true
	spec.Memory.Limit = math.MaxUint64
	spec.Memory.SwapLimit = math.MaxUint64
	if config.Cgroups.Memory > 0 {
		spec.Memory.Limit = uint64(config.Cgroups.Memory)
	}
	if config.Cgroups.MemorySwap > 0 {
		spec.Memory.SwapLimit = uint64(config.Cgroups.MemorySwap)
	}

	// Get CPU info
	spec.HasCpu = true
	spec.Cpu.Limit = 1024
	if config.Cgroups.CpuShares != 0 {
		spec.Cpu.Limit = uint64(config.Cgroups.CpuShares)
	}
	spec.Cpu.Mask = utils.FixCpuMask(config.Cgroups.CpusetCpus, mi.NumCores)

	spec.HasNetwork = true
	spec.HasDiskIo = true

	return spec
}
```

위 항목 모두 libcontainer의 api를 통해 가져오며 libcontainer는 cgroup config를 읽어 가져온다.

### CPU

```
type ContainerInfo struct {
	ContainerReference

	// The direct subcontainers of the current container.
	Subcontainers []ContainerReference `json:"subcontainers,omitempty"`

	// The isolation used in the container.
	Spec ContainerSpec `json:"spec,omitempty"`

	// Historical statistics gathered from the container.
	Stats []*ContainerStats `json:"stats,omitempty"`
}
```

```
type ContainerStats struct {
	// The time of this stat point.
	Timestamp time.Time    `json:"timestamp"`
	Cpu       CpuStats     `json:"cpu,omitempty"`
	DiskIo    DiskIoStats  `json:"diskio,omitempty"`
	Memory    MemoryStats  `json:"memory,omitempty"`
	Network   NetworkStats `json:"network,omitempty"`

	// Filesystem statistics
	Filesystem []FsStats `json:"filesystem,omitempty"`

	// Task load stats
	TaskStats LoadStats `json:"task_stats,omitempty"`
}
```

```
func (s *CpuacctGroup) GetStats(path string, stats *cgroups.Stats) error {
	userModeUsage, kernelModeUsage, err := getCpuUsageBreakdown(path)
	if err != nil {
		return err
	}

	totalUsage, err := getCgroupParamUint(path, "cpuacct.usage")
	if err != nil {
		return err
	}

	percpuUsage, err := getPercpuUsage(path)
	if err != nil {
		return err
	}

	stats.CpuStats.CpuUsage.TotalUsage = totalUsage
	stats.CpuStats.CpuUsage.PercpuUsage = percpuUsage
	stats.CpuStats.CpuUsage.UsageInUsermode = userModeUsage
	stats.CpuStats.CpuUsage.UsageInKernelmode = kernelModeUsage
	return nil
}
```

```
kyu@kyu-HP-EliteBook-2570p:/sys/fs/cgroup/cpuacct$ ls -al
합계 0
drwxr-xr-x  4 root root   0  4월 22 16:16 .
drwxr-xr-x 12 root root 240  4월 22 16:16 ..
-rw-r--r--  1 root root   0  4월 22 16:16 cgroup.clone_children
--w--w--w-  1 root root   0  4월 22 16:16 cgroup.event_control
-rw-r--r--  1 root root   0  4월 22 16:16 cgroup.procs
-r--r--r--  1 root root   0  4월 22 16:16 cgroup.sane_behavior
-r--r--r--  1 root root   0  4월 22 16:16 cpuacct.stat
-rw-r--r--  1 root root   0  4월 22 16:16 cpuacct.usage
-r--r--r--  1 root root   0  4월 22 16:16 cpuacct.usage_percpu
drwxr-xr-x  4 root root   0  4월 25 16:55 docker
-rw-r--r--  1 root root   0  4월 22 16:16 notify_on_release
-rw-r--r--  1 root root   0  4월 22 16:16 release_agent
-rw-r--r--  1 root root   0  4월 22 16:16 tasks
drwxr-xr-x  4 root root   0  4월 22 16:16 user
```

### Memory

```
func (s *MemoryGroup) GetStats(path string, stats *cgroups.Stats) error {
	// Set stats from memory.stat.
	statsFile, err := os.Open(filepath.Join(path, "memory.stat"))
	if err != nil {
		if os.IsNotExist(err) {
			return nil
		}
		return err
	}
	defer statsFile.Close()

	sc := bufio.NewScanner(statsFile)
	for sc.Scan() {
		t, v, err := getCgroupParamKeyValue(sc.Text())
		if err != nil {
			return fmt.Errorf("failed to parse memory.stat (%q) - %v", sc.Text(), err)
		}
		stats.MemoryStats.Stats[t] = v
	}

	// Set memory usage and max historical usage.
	value, err := getCgroupParamUint(path, "memory.usage_in_bytes")
	if err != nil {
		return fmt.Errorf("failed to parse memory.usage_in_bytes - %v", err)
	}
	stats.MemoryStats.Usage = value
	value, err = getCgroupParamUint(path, "memory.max_usage_in_bytes")
	if err != nil {
		return fmt.Errorf("failed to parse memory.max_usage_in_bytes - %v", err)
	}
	stats.MemoryStats.MaxUsage = value
	value, err = getCgroupParamUint(path, "memory.failcnt")
	if err != nil {
		return fmt.Errorf("failed to parse memory.failcnt - %v", err)
	}
	stats.MemoryStats.Failcnt = value

	return nil
}
```

```
kyu@kyu-HP-EliteBook-2570p:/sys/fs/cgroup/memory$ ls -al
합계 0
drwxr-xr-x  4 root root   0  4월 22 16:16 .
drwxr-xr-x 12 root root 240  4월 22 16:16 ..
-rw-r--r--  1 root root   0  4월 22 16:16 cgroup.clone_children
--w--w--w-  1 root root   0  4월 22 16:16 cgroup.event_control
-rw-r--r--  1 root root   0  4월 22 16:16 cgroup.procs
-r--r--r--  1 root root   0  4월 22 16:16 cgroup.sane_behavior
drwxr-xr-x  4 root root   0  4월 25 16:55 docker
-rw-r--r--  1 root root   0  4월 22 16:16 memory.failcnt
--w-------  1 root root   0  4월 22 16:16 memory.force_empty
-rw-r--r--  1 root root   0  4월 22 16:16 memory.kmem.failcnt
-rw-r--r--  1 root root   0  4월 22 16:16 memory.kmem.limit_in_bytes
-rw-r--r--  1 root root   0  4월 22 16:16 memory.kmem.max_usage_in_bytes
-r--r--r--  1 root root   0  4월 22 16:16 memory.kmem.slabinfo
-rw-r--r--  1 root root   0  4월 22 16:16 memory.kmem.tcp.failcnt
-rw-r--r--  1 root root   0  4월 22 16:16 memory.kmem.tcp.limit_in_bytes
-rw-r--r--  1 root root   0  4월 22 16:16 memory.kmem.tcp.max_usage_in_bytes
-r--r--r--  1 root root   0  4월 22 16:16 memory.kmem.tcp.usage_in_bytes
-r--r--r--  1 root root   0  4월 22 16:16 memory.kmem.usage_in_bytes
-rw-r--r--  1 root root   0  4월 22 16:16 memory.limit_in_bytes
-rw-r--r--  1 root root   0  4월 22 16:16 memory.max_usage_in_bytes
-rw-r--r--  1 root root   0  4월 22 16:16 memory.move_charge_at_immigrate
-r--r--r--  1 root root   0  4월 22 16:16 memory.numa_stat
-rw-r--r--  1 root root   0  4월 22 16:16 memory.oom_control
----------  1 root root   0  4월 22 16:16 memory.pressure_level
-rw-r--r--  1 root root   0  4월 22 16:16 memory.soft_limit_in_bytes
-r--r--r--  1 root root   0  4월 22 16:16 memory.stat
-rw-r--r--  1 root root   0  4월 22 16:16 memory.swappiness
-r--r--r--  1 root root   0  4월 22 16:16 memory.usage_in_bytes
-rw-r--r--  1 root root   0  4월 22 16:16 memory.use_hierarchy
-rw-r--r--  1 root root   0  4월 22 16:16 notify_on_release
-rw-r--r--  1 root root   0  4월 22 16:16 release_agent
-rw-r--r--  1 root root   0  4월 22 16:16 tasks
drwxr-xr-x  4 root root   0  4월 22 16:16 user
```
### Network

```
func getNetworkStats(name string, sysFs sysfs.SysFs) (info.NetworkStats, error) {
	stats := info.NetworkStats{}
	var err error
	stats.RxBytes, err = sysFs.GetNetworkStatValue(name, "rx_bytes")
	if err != nil {
		return stats, err
	}
	stats.RxPackets, err = sysFs.GetNetworkStatValue(name, "rx_packets")
	if err != nil {
		return stats, err
	}
	stats.RxErrors, err = sysFs.GetNetworkStatValue(name, "rx_errors")
	if err != nil {
		return stats, err
	}
	stats.RxDropped, err = sysFs.GetNetworkStatValue(name, "rx_dropped")
	if err != nil {
		return stats, err
	}
	stats.TxBytes, err = sysFs.GetNetworkStatValue(name, "tx_bytes")
	if err != nil {
		return stats, err
	}
	stats.TxPackets, err = sysFs.GetNetworkStatValue(name, "tx_packets")
	if err != nil {
		return stats, err
	}
	stats.TxErrors, err = sysFs.GetNetworkStatValue(name, "tx_errors")
	if err != nil {
		return stats, err
	}
	stats.TxDropped, err = sysFs.GetNetworkStatValue(name, "tx_dropped")
	if err != nil {
		return stats, err
	}
	return stats, nil
}
```

```
func (self *realSysFs) GetNetworkStatValue(dev string, stat string) (uint64, error) {
	statPath := path.Join(netDir, dev, "/statistics", stat)
	out, err := ioutil.ReadFile(statPath)
	if err != nil {
		return 0, fmt.Errorf("failed to read stat from %q for device %q", statPath, dev)
	}
	var s uint64
	n, err := fmt.Sscanf(string(out), "%d", &s)
	if err != nil || n != 1 {
		return 0, fmt.Errorf("could not parse value from %q for file %s", string(out), statPath)
	}
	return s, nil
}
```

```
kyu@kyu-HP-EliteBook-2570p:/sys/class/net$ ls -al
합계 0
drwxr-xr-x  2 root root 0  4월 23 01:16 .
drwxr-xr-x 65 root root 0  4월 23 01:16 ..
lrwxrwxrwx  1 root root 0  4월 22 16:16 br-tun -> ../../devices/virtual/net/br-tun
lrwxrwxrwx  1 root root 0  4월 22 17:09 docker0 -> ../../devices/virtual/net/docker0
lrwxrwxrwx  1 root root 0  4월 22 16:16 eth0 -> ../../devices/pci0000:00/0000:00:19.0/net/eth0
lrwxrwxrwx  1 root root 0  4월 23 01:16 lo -> ../../devices/virtual/net/lo
lrwxrwxrwx  1 root root 0  4월 22 16:16 ovs-system -> ../../devices/virtual/net/ovs-system
lrwxrwxrwx  1 root root 0  4월 25 15:58 veth4113c31 -> ../../devices/virtual/net/veth4113c31
lrwxrwxrwx  1 root root 0  4월 26 01:05 vethd5143b2 -> ../../devices/virtual/net/vethd5143b2
lrwxrwxrwx  1 root root 0  4월 22 16:16 virbr0 -> ../../devices/virtual/net/virbr0
lrwxrwxrwx  1 root root 0  4월 22 16:16 wlan0 -> ../../devices/pci0000:00/0000:00:1c.3/0000:24:00.0/net/wlan0
```

```
kyu@kyu-HP-EliteBook-2570p:~$ netstat -i
Kernel Interface table
Iface   MTU Met   RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
br-tun     1500 0         0      0      0 0             8      0      0      0 BRU
docker0    1500 0   1193026      0      0 0       2093062      0      0      0 BMRU
eth0       1500 0  53237347      0     48 0       5010292      0      0      0 BMRU
lo        65536 0      1124      0      0 0          1124      0      0      0 LRU
veth4113c31  1500 0    223532      0      0 0        285284      0      0      0 BRU
virbr0     1500 0         0      0      0 0             0      0      0      0 BMU
```

### Filesystem

```
func (self *realSysFs) GetBlockDevices() ([]os.FileInfo, error) {
	return ioutil.ReadDir(blockDir)
}

func (self *realSysFs) GetBlockDeviceNumbers(name string) (string, error) {
	dev, err := ioutil.ReadFile(path.Join(blockDir, name, "/dev"))
	if err != nil {
		return "", err
	}
	return string(dev), nil
}
```

```
yu@kyu-HP-EliteBook-2570p:/sys/block$ ls -al
합계 0
drwxr-xr-x  2 root root 0  4월 23 01:16 .
dr-xr-xr-x 13 root root 0  4월 23 01:16 ..
lrwxrwxrwx  1 root root 0  4월 22 16:16 loop0 -> ../devices/virtual/block/loop0
lrwxrwxrwx  1 root root 0  4월 22 16:16 loop1 -> ../devices/virtual/block/loop1
lrwxrwxrwx  1 root root 0  4월 22 16:16 loop2 -> ../devices/virtual/block/loop2
lrwxrwxrwx  1 root root 0  4월 22 16:16 loop3 -> ../devices/virtual/block/loop3
lrwxrwxrwx  1 root root 0  4월 22 16:16 loop4 -> ../devices/virtual/block/loop4
lrwxrwxrwx  1 root root 0  4월 22 16:16 loop5 -> ../devices/virtual/block/loop5
lrwxrwxrwx  1 root root 0  4월 22 16:16 loop6 -> ../devices/virtual/block/loop6
lrwxrwxrwx  1 root root 0  4월 22 16:16 loop7 -> ../devices/virtual/block/loop7
lrwxrwxrwx  1 root root 0  4월 22 16:16 ram0 -> ../devices/virtual/block/ram0
lrwxrwxrwx  1 root root 0  4월 22 16:16 ram1 -> ../devices/virtual/block/ram1
lrwxrwxrwx  1 root root 0  4월 22 16:16 ram10 -> ../devices/virtual/block/ram10
lrwxrwxrwx  1 root root 0  4월 22 16:16 ram11 -> ../devices/virtual/block/ram11
lrwxrwxrwx  1 root root 0  4월 22 16:16 ram12 -> ../devices/virtual/block/ram12
lrwxrwxrwx  1 root root 0  4월 22 16:16 ram13 -> ../devices/virtual/block/ram13
lrwxrwxrwx  1 root root 0  4월 22 16:16 ram14 -> ../devices/virtual/block/ram14
lrwxrwxrwx  1 root root 0  4월 22 16:16 ram15 -> ../devices/virtual/block/ram15
lrwxrwxrwx  1 root root 0  4월 22 16:16 ram2 -> ../devices/virtual/block/ram2
lrwxrwxrwx  1 root root 0  4월 22 16:16 ram3 -> ../devices/virtual/block/ram3
lrwxrwxrwx  1 root root 0  4월 22 16:16 ram4 -> ../devices/virtual/block/ram4
lrwxrwxrwx  1 root root 0  4월 22 16:16 ram5 -> ../devices/virtual/block/ram5
lrwxrwxrwx  1 root root 0  4월 22 16:16 ram6 -> ../devices/virtual/block/ram6
lrwxrwxrwx  1 root root 0  4월 22 16:16 ram7 -> ../devices/virtual/block/ram7
lrwxrwxrwx  1 root root 0  4월 22 16:16 ram8 -> ../devices/virtual/block/ram8
lrwxrwxrwx  1 root root 0  4월 22 16:16 ram9 -> ../devices/virtual/block/ram9
lrwxrwxrwx  1 root root 0  4월 23 01:16 sda -> ../devices/pci0000:00/0000:00:1f.2/ata1/host0/target0:0:0/0:0:0:0/block/sda
```

```
kyu@kyu-HP-EliteBook-2570p:/dev/disk/by-uuid$ ls -al
합계 0
drwxr-xr-x 2 root root 100  4월 23 01:16 .
drwxr-xr-x 5 root root 100  4월 23 01:16 ..
lrwxrwxrwx 1 root root  10  4월 22 16:16 008244d8-3cf5-40f2-97fc-45a1972ed6bc -> ../../sda6
lrwxrwxrwx 1 root root  10  4월 22 16:16 9EC66000C65FD6DD -> ../../sda1
lrwxrwxrwx 1 root root  10  4월 22 16:16 b9348159-88cd-4b01-9def-59417c15471a -> ../../sda5
```
