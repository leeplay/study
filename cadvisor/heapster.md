Overview
========

Heapster enables Container Cluster Monitoring.
Internally, heapster uses cAdvisor for compute resource usage metrics.

Heapster currently supports Kubernetes and CoreOS natively. It can be extended to support other cluster management solutions easily. While running in a Kube cluster, heapster collects compute resource usage of all pods and nodes.

Source configuration is documented here.
Running Heapster on Kubernetes

Heapster supports a pluggable storage backend. It supports InfluxDB with Grafana, Google Cloud Monitoring and Google Cloud Logging. We welcome patches that add additional storage backends.

To run Heapster on a Kubernetes cluster with,

    InfluxDB use this guide.
    Google Cloud Monitoring and Google Cloud Logging use this guide.

Take a look at the storage schema here.

heapster는 resource usage metrics를 계산하기 위해 cAdvisor를 사용해 container cluster 모니터링을 가능하게 함


http://rancher.com/comparing-monitoring-options-for-docker-deployments/

http://rancher.com/docker-monitoring-continued-prometheus-and-sysdig/
