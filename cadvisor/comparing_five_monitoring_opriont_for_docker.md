## Comparing Five Monitoring Options for Docker

## Docker Stats

- docker 1.5.0에서 기본 커맨드로 컨테이너의 리소스 사용량을 알려주는 명령어가 추가
- 각 컨테이너의 cpu 이용량, 사용 중인 메모리와 전체 메모리, 네트워크 전송량 정보를 제공
- docker api를 직접 제공해 더 자세한 stat 정보를 얻을 수 있음

```
$ docker stats determined_shockley determined_wozniak prickly_hypatia
CONTAINER             CPU %               MEM USAGE/LIMIT       MEM %               NET I/O
determined_shockley   0.00%               884 KiB/1.961 GiB     0.04%               648 B/648 B
determined_wozniak    0.00%               1.723 MiB/1.961 GiB   0.09%               1.266 KiB/648 B
prickly_hypatia       0.00%               740 KiB/1.961 GiB     0.04%               1.898 KiB/648 B
```

## CAdvisor

The docker stats command and the remote API are useful for getting information on the command line, however, if you would like to access the information in a graphical interface you will need a tool such as CAdvisor. CAdvisor provides a visual representation of the data shown by the docker stats command earlier.  Run the docker command below and go to http://<your-hostname>:8080/ in the browser of your choice to see the CAdvisor interface. You will be shown graphs for overall CPU usage, Memory usage, Network throughput and disk space utilization. You can then drill down into the usage statistics for a specific container by clicking the Docker Containers link at the top of the page and then selecting the container of your choice.  In addition to these statistics CAdvisor also shows the limits, if any, that are placed on container, using the Isolation section.

CAdvisor is a useful tool that is trivially easy to setup, it saves us from having to ssh into the server to look at resource consumption and also produces graphs for us. In addition the pressure gauges provide a quick overview of when a your cluster needs additional resources. Furthermore, unlike other options in this article CAdvisor is free as it is open source and also it runs on hardware already provisioned for your cluster, other than some processing resources there is no additional cost of running CAdvisor. However, it has it limitations; it can only monitor one docker host and hence if you have a multi-node deployment  your stats will be disjoint and spread though out your cluster. Note that you can use heapster to monitor multiple nodes if you are running Kubernetes.  The data in the charts is a moving window of one minute only and there is no way to look at longer term trends. There is no mechanism to kick-off alerting if the resource usage is at dangerous levels. If you currently do not have any visibility in to the resource consumption of your docker node/cluster then CAdvisor is a good first step into container monitoring however, if you intend to run any critical tasks on your containers a more robust tool or approach is needed.  Note that Rancher runs CAdvisor on each connected host, and exposes a limited set of stats through the UI, and all of the system stats through the API.

## Score Card

- 5점 만점

|          |docker|cadvisor|scout|Data Dog|Sensu|
|----------|------|--------|-----|--------|-----|
|쉬운 배포|5|5| | | |
|detail 레벨|5|2| | | |
|aggregation 레벨|0|1| | | |
|alert 지원|0|0| | | |
|non-docker 지원|0|0| | | |
|비용|free|free| | | |

## conclusion



## 출처

- http://rancher.com/comparing-monitoring-options-for-docker-deployments/
