kubernetes 동작원리
===================


> 단일 서버에서 컨테이너를 사용한다면 쿠버네티스를 사용하지 않아도 되지만

> 서버가 2대 이상으로 늘어나게 된다면, 어떤 서버에 어떤 컨테이너를 생성해야 할지 서버마다 들어가서 

> 모든 작업을 각각의 서버에 접근해서 수행해야 하는 불편한 상황이 생긴다.

> 이러한 문제를 해결하기 위해 컨테이너 오케스트레이션이 등장

## 쿠버네티스 아키텍쳐

![image](https://user-images.githubusercontent.com/94096054/153351920-d72e29e9-6dfa-475b-ba38-05937e575b84.png)

> 쿠버네티스 클러스터는 크게 Control plane과 Worker node로 구성된다.

1. Control Plane(Master node)

> 마스터 노드는 클러스터를 통제하고 관리하는 쿠버네티스의 중심부로 아래와 같은 구성요소가 있다.

* API Server
  
  > 클라이언트와 통신하는 쿠버네티스 API 서버로 쿠버네티스 구성요소들간의 통신은 대부분 API 서버를 거친다.

* Scheduler

  > API 서버가 노드들에게 일을 할당할 때 직접 시키는 게 아니라 스케줄러에게 시킨다.

  > 스케줄러는 애플리케이션을 노드에 할당하는 기능을 한다.

* etcd 

  > 분산 저장소, 안드로이드에서 자주 사용되는 sqlite를 사용하기도 함

  > 선언형 api를 저장하고 controller가 지속적으로 관리 

* Controller Manager

  > 쿠버네티스는 선언형 api를 사용

  > Controller Manager의 요소들은 선언된 API에 맞게 쿠버네티스 resource들을 생성, 복제, 관리한다.

  > 쿠버네티스의 비즈니스 로직은 모두 여기에 저장되어있다. 

2. Worker Node

* Kubelet

  > 스케줄러에서 워커노드들에게 애플리케이션을 할당할 때 kubelet에 요청

  > kubelet은 pod에서 컨테이너가 동작하도록 하고, pod을 관리

* kube-proxy
  
  > 네트워크 관련 규칙을 유지해주는 곳


## Kubernetes 동작원리 

![image](https://user-images.githubusercontent.com/94096054/153352529-78821d21-db05-4e8b-9d3e-705e694541d0.png)

> 쿠버네티스는 선언형 API를 사용한다.

* 명령형 API 
  
  > 일반적으로 사용하는 CLI 명령어와 동일

* 선언형 API 

  > 원하는 결과만을 제시하고 시스템이 스스로 결과를 얻어주는 API

> 선언형 API를 사용하여 내용이 etcd에 저장되고, etcd를 모니터링 하는 controller에 의해 동작하게 된다.

## Kubernetes 동작순서

1. 클라이언트는 쿠버네티스 API Server에 요청

> curl이나 postman 등을 사용하여 http request를 보낼 수도 있지만

> 일반적으로 kubectl을 사용하여 요청

2. etcd에 API의 내용 저장

> API Server에 선언형 api로 요청이 들어오면 선언형 api의 특징답게 바로 컨테이너를 생성하라는 명령을 동작시키지 않는다.

> 분산 저장소인 etcd에 들어온 내용을 저장한다.

3. etcd를 감시하던 controller 동작

> controller는 etcd에 내가 담당하고 있는 resource가 들어왔는지를 감시하다가 

> etcd에 자신의 역할에 맞는 내용이 저장되어 있다면 스케줄러에게 동작을 요청

> 쿠버네티스의 비즈니스 로직은 controller에 숨어있다.

4. 스케줄러는 동작

> 스케줄러는 워커노드의 kubelet과 통신

5. kubelet 동장

> kubelet은 노드에 pod 등을 생성




