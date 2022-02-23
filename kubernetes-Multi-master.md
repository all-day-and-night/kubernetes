Multi-master
=============

## 쿠버네티스 클러스터를 직접 구성하는 도구

* kubeadm
> 쿠버네티스에서 공식 제공하는 클러스터 생성/관리 도구

* kubespray
  + 쿠버네티스 클러스터를 배포하는 오픈소스 프로젝트

  + 다양한 형식으로 쿠버네티스 클러스터 구성 가능

  + 온프레미스에서 상용 서비스 클러스터 운영 시 유용

* Control plane(master node)

  + 워커 노드들의 상태를 관리하고 제어
  + Highly Available(HA) cluster 운영
  + api는 loadbalancer를 통해 worker node에 노출
  + 최소 3개의 중첩된 control plane을 구성(5, 7개의 master nodes)

* worker node
  + 도커 플랫폼을 통해 컨테이너를 동작하며 실제 서비스 제공

## multi-master(HA) 클러스터 구성 flow

![image](https://user-images.githubusercontent.com/94096054/155273379-68aea46e-f4ef-4393-a578-92e52f36f222.png)


> 각 마스터 노드는 loadbalance를 통해 하나의 master node와 연결하고

> 각 마스터 노드의 etcd를 동기화하기 위해 api로 데이터를 전송한다(모든 master node는 같은 데이터를 갖고 있다)

> kubelet은 master node와 연동하는 것이 아니라 load balancer에 데이터를 전송한다.

> 마스터 노드가 강제 종료되어도 다른 마스터 노드를 실행하는 고가용성 서버를 구축하는 것이 목표


## Highly Available cluster 구성 순서 

1. all system - runtime(Docker) install

2. control plane, worker node -kubeadm 설치

  + 설치 전 환경설정
  + kubeadm, kubectl, kubelet 설치

3. LB(load balancer) 구성

4. kubeadm을 이용한 HA 클러스터 구성

  + master1: kubeadm init 명령으로 초기화 -LB 등록
  + master2, master3을 master1에 조인
  + CNI(Container Network interface) 설치
  + worker node를 LB 통해 master와 join
  + 설치된 시스템 확인


## 설치 

1. 환경 설정

> load balancer, # of master node, # of worker node의 개수에 따라 서버 구축

2. 컨테이너 runtime 설치

> runtime으로 도커를 설치하자

```
yum
```
......

> 일단 설치는 나중에 하자 

https://www.youtube.com/watch?v=b457Nrk9GKk&list=PLApuRlvrZKohaBHvXAOhUD-RxD0uQ3z0c&index=24


















