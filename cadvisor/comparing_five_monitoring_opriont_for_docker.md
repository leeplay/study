## Comparing Five Monitoring Options for Docker

## Docker Stats

The first tool I will talk about is Docker itself, 
yes you may not be aware that docker client already provides a rudimentary command line tool to inspect containers’
resource consumption. To look at the container stats run docker stats with the name(s) of the running container(s)
for which you would like to see stats. This will present the CPU utilization for each container, 
the memory used and total memory available to the container. 
Note that if you have not limited memory for containers this command will post total memory of your host. 
This does not mean each of your container has access to that much memory. 
In addition you will also be able to see total data sent and received over the network by the container.

처음으로 소개할 툴은 Docker 입니다. 
docker client에서 이미 기본 커맨드로 컨테이너의 리소스 사용량을 알려주는 명령어가 있습니다.
컨테이너 stats 정보를 확인하려면 docker stats container-name 을 실행하면 됩니다. 각 컨테이너의 cpu 이용량,
사용 중인 메모리와 전체 메모리 정보를 제공합니다. 
만약 컨테이너에서 사용하는 메모리를 제한하지 않았다면 이 커맨드는 호스트의 전체 메모리를 알려줍니다, 이 말은 각 컨테이너가 접근할 수 있는 메모리를 의미하지 않습니다. 
추가적으로 컨테이너의 네트워크 송수신 데이터의 양을 확인할 수 있습니다. 

```
$ docker stats determined_shockley determined_wozniak prickly_hypatia
CONTAINER             CPU %               MEM USAGE/LIMIT       MEM %               NET I/O
determined_shockley   0.00%               884 KiB/1.961 GiB     0.04%               648 B/648 B
determined_wozniak    0.00%               1.723 MiB/1.961 GiB   0.09%               1.266 KiB/648 B
prickly_hypatia       0.00%               740 KiB/1.961 GiB     0.04%               1.898 KiB/648 B
```

더 자세한 stats를 얻고 싶다면 리눅스 툴들을 활용해 api를 직접 설계해야 한다.

## Score Card

- 5점 만점, O는 지원, X는 지원 안함

|          |docker|cadvisor|scout|Data Dog|Sensu|
|----------|------|--------|-----|--------|-----|
|쉬운 배포|5| | | | |
|detail 레벨|5| | | | |
|aggregation 레벨|X| | | | |
|alert 지원|X| | | | |
|non-docker 지원|X| | | | |

## conclusion



## 출처

- http://rancher.com/comparing-monitoring-options-for-docker-deployments/
