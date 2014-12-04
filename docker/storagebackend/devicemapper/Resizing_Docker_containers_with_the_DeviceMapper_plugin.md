Resizing Docker containers with the Device Mapper plugin
========================================================

If you're using Docker on CentOS, RHEL, Fedora, or any other distro that doesn't ship by default with AUFS support, you are probably using the Device Mapper storage plugin. 

만약 기본으로 AUFS를 지원하지 않는 centos, rhel, fedora 또는 다른 리눅스 배포판에서 도커를 사용한다면 DeviceMapper 스토리지 플러그인를 사용할 것이다.  

By default, this plugin will store all your containers in a 100 GB sparse file, and each container will be limited to 10 GB. This article will explain how you can change that limit, and move container storage to a dedicated partition or LVM volume.

기본으로 이 플러그인 모든 컨테이너를 100기가의 sparse 파일로 저장하게 됩니다. 그리고 각 컨테이너는 10GB로 제한되어 사용됩니다. 사이즈 제한을 변경하는 방법과 컨테이너 저장소를 dedecated 파티션 또는 LVM volume으로 옮기는 방법을 소개하겠습니다.


###How it works

To really understand what we're going to do, let's look how the Device Mapper plugin works.

이해를 돕기 위해 DeviceMapper 플러그인의 동작 방법을 보겠습니다.

It is based on the Device Mapper "thin target". It's actually a snapshot target, but it is called "thin" because it allows thin provisioning. Thin provisioning means that you have a (hopefully big) pool of available storage blocks, and you create block devices (virtual disks, if you will) of arbitrary size from that pool; but the blocks will be marked as used (or "taken" from the pool) only when you actually write to it.

"thin target"은 DeviceMapper의 기본입니다. 실제로 Snapshot target입니다. thin provisioning을 허용하기 때문에 thin으로 불립니다. thin provisioning은 사용할 수 있는 storage blocks과 block devices를 통해 당신이 희망하는 만큼 임의의 사이즈로 pool을 할당하는 것을 의미합니다. 그러나 blocks은 실제로 디스크에 기록할 때 쓰여집니다.  

This means that you can oversubscribe the pool; e.g. create thousands of 10 GB volumes with a 100 GB pool, or even a 100 TB volume on a 1 GB pool. As long as you don't actually write more blocks than you actually have in the pool, everything will be fine.

pool의 용량 이상으로 크기를 가질 수 있다. 예를들어 100GB 풀에서 10GB volume을 100개 생성하거나 설사 1GB 풀에서 100TB 생성할 수 있다. 실제로 pool 많은 블록을 기록하지 않는다. 실제 풀의 용량보다 많은 블록을 사용하지 않는다면 모든 것이 괜찮습니다.

Additionally, the thin target is able to perform snapshots. It means that at any time, you can create a shallow copy of an existing volume. From a user point of view, it's exactly as if you now had two identical volumes, that can be changed independently. As if you had made a full copy, except that it was instantaneous (even for large volumes), and they don't use twice the storage. Additional storage is used only when changes are made in one of the volumes. Then the thin target allocates new blocks from the storage pool.

추가로 thin target은 snapshot을 가능하게 합니다. 기존 volume의 단순 복사본을 만들 수 있다는 뜻입니다. 
사용자 관점에서 보면 마치 독립적으로 변경될 수 있는 두 개의 동일한 볼륨을 가지고 있는 것과 같습니다. 마치 당신이 full copy를 한 것처럼 말이죠 그리고 storage를 두 번 사용하지 않고 하나의 voluems에서 변경이 발생하는 경우 추가 storage가 사용됩니다. 그 다음 thin target은 storage pool에 새로운 블록을 할당합니다. 

