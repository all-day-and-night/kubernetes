kubernetes-intro
==================


## Kubernetes

> 컨테이너를 도커 플랫폼에 올려서 관리 + 운영 + 클러스터 서비스 지원



## kubectl

* kubectl [command] [TYPE] [NAME] [FLAGS]

  + command 
  
  > 자원(object)에 실행할 명령

  > create, get, delete, edit

  + TYPE
  
  > 자원의 타입

  > node, pod, service

  * NAME

  > 자원의 이름

  * FLAGS

  > 부가적으로 설정할 옵션


## kubectl 명령어 정리

* kubectl api-resources 

> object 

* kubectl get nodes

> node에 대한 정보 출력

* kubectl --help

> 쿠버네티스의 도움말 출력

> command, object, type, flags의 정보를 얻을 수 있음

* kubectl describe [object] [node name]

> object에 대한 정보 출력


* kubectl run [name] --image=nginx:1.14 --port 80

> pod 생성 















