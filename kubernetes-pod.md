POD
=====


* Container

> 하나의 애플리케이션 


## POD

> 컨테이너를 표현하는 K8S api의 최소 단위 

> Pod에는 하나 도는 여러 개의 컨테이너가 포함될 수 있음


* Pod 생성하기

  * CLI
  
  > kubectl run webserver --image=nginx:1.14
  
  * yaml

  > apiVersion....


* Pod 실행

```
kubectl create -f pod-nginx.yaml
```
