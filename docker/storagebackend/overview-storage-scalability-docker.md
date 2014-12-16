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

대안을 찾자
===========

