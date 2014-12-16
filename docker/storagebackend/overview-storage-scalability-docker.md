배경
====

처음에는 2013년 초 오픈소스 이후 도커의 storage 상황에 대한 간략한 뒷이야기입니다. 
이 당시 도커는 AUFS라는 파일시스템을 사용했습니다. 

유니온 파일시스템은 도커의 필요한 기능 몇 가지를 지원합니다.

- container creation speed
- copy-on-write image->container

Docker still supports the AUFS backend, but Ubuntu has disabled it and moved the AUFS kernel module to 
linux-image-extra.  The fact that AUFS never made it into the upstream Linux kernel poses a problem for Red Hat, 
where the policy is upstream first, and, out-of-tree bits are not included. 
Of course, that doesn’t preclude experiments of all shapes and sizes!

도커는 여전히 AUFS를 지원합니다. 그러나 우분투 AUFS는 비활성화 되고 있고 AUFS커널 모듈은 linux-image-extra로 옮겼습니다. 
이 사실로 AUFS 업스트림 리눅스 커널에 포함되지 않았다는 문제가 제기되었습니다.
정책은 업스트림이 최우선이며 리눅스 커널 트리에 제거된 건 포함하지 않습니다.
물론 모든 형태와 크기의 경험이 불가능하진 않습니다. 

대안을 찾자
===========

우리는 최신, 안전, 유지보수, 오랜기간 지원 그리고 성능이 뛰어난 AUFS 대안이 필요하다는 것을 알았습니다. 
흥미롭게도 위의 기준을 만족하는 해결책이 레드햇 커널 엔지니어들이 만들어놨습니다. device mapper thin provisioning이죠
몇 몇 레드햇 엔지니어들이 도커를 위해 새로운 storage를 만들었고 docker 0.7 버전에 포함되었습니다. 페도라, 센토스 또는 다른 RHEL 에서 도커를 사용하면 디폴트로 loopback에 마운트한 sparse file을 사용하는 device-mapper를 사용합니다. 

Devicemapper thin provisioning을 더한 마운트된 루프백 디바이스는 아무런 설정없이 생산성 저하없이 간단하게 도커 설치와 시작을 할 수 있는 기존 방식을 유지할 수 있었습니다. 이 기능은 다커의 중요한 승리 요인 중 하나 입니다.  




