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

