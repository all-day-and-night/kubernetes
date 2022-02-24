Service
=========

> 쿠버네티스 네트워크

1. 서비스 개념
2. 서비스 타입
3. 서비스 사용하기
4. 헤드리스 서비스
5. kube-proxy(load balance)

## kubernetes Service 개념

> 동일한 서비스를 제공하는 Pod 그룹의 "단일 진입점" 제공

> Service api를 사용하여 실행한 pod들을 하나의 IP로 묶어서 관리

> pod의 key:value로 묶어서 관리

> 작업을 균등하게 Pod에 분배하여 애플리케이션 제공


* Deployment-definition
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webui
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webui
  template:
    metadata:
      name: nginx-pod
      labels:
        app: webui
    spec:
      containers:
      - name: nginx-container
        image: nginx:1.14
```

* Service-definition
```
apiVersion: v1
kind: Service
metadata:
  name: webui-svc
spec:
  # load balancer IP
  clusterIP: 10.96.100.100
  selector:
    app: webui
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  ```

## Service Type

1. clusterIP

> Pod 그룹의 단일 진입점(virtual IP) 생성

2. NodePort

> Cluster IP 생성 후

> 모든 worker Node에 외부에서 접속 가능한 포트 예약

3. LoadBalancer

> 클라우드 인프라스트럭쳐(AWS, Azure, GCP)나 오픈스택 클라우드에 적용

> LoadBalancer를 자동으로 비전하는 기능 지원

4. ExternalName

> 클러스터 안에서 외부 접속시 사용할 도메인을 등록해서 사용

> 클러스터 도메인이 실제 외부 도메인으로 치환되어 동작


## Service 4가지 실습


1. ClusterIP

  + selector의 label이 동일한 Pod들의 그룹으로 묶어

  + 단일 진입점(virtual ip) 생성

  + 클러스터 내부에서만 사용가능

  + type 생략시 default 값으로 10.96.0.0/12 범위에서 할당

* clusterIP Service
```
apiVersion: v1
kind: Service
metadata:
  name: clusterip-service
spec:
  type: ClusterIP
  clusterIP: 10.100.100.100
  selector:
    app: webui
  ports:
  -protocol: TCP
    port: 80
    targetPort: 80
```

2. NodePort

* 모든 노드를 대상으로 외부 접속 가능한 포트를 예약
* Default NodePort 범위: 30000-32767
* ClusterIP 생성 후 NodePort 예약

* NodePort Service
```
apiVersion: v1
kind: Service
metadata:
  name: clusterip-service
spec:
  type: NodePort
  clusterIP: 10.100.100.200
  selector:
    app: webui
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    ## NodePort
    nodePort: 30200
```

3. LoadBalancer

* Public cloud(AWS, Azure, GCP 등) 에서 운영가능 
* LoadBalancer를 자동으로 구성 요청
* NodePort를 예약 후 해당 nodeport로 외부 접근을 허용

* LoadBalancer Service - definition

```
apiVersion: v1
kind: Service
metadata:
  name: loadbalancer-service
spec:
  type: LoadBalancer
  selector:
    app: webui
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

4. ExternalName

* 클러스터 내부에서 External(외부)의 도메인을 설정
* DNS 제공

* ExternalName Service-definition

```
apiVersion: v1
kind: Service
metadata:
  name: externalname-svc
spec:
  type: ExternalName
  externalName: google.com
```

5. Kubernetes 헤드리스 서비스


* Cluster IP가 없는 서비스로 단일 진입점이 필요없을 때 사용
* Service와 연결된 Pod의 endPoint로 DNS 레코드가 생성됨
* Pod의 DNS 주소 : pod-ip-addr.namespace.pod.cluster.local


> Pod들의 endPoint에 DNS resolving Service 지원

* headless Sevice-definition
```
apiVersion: v1
kind: Service
metadata:
  name: headless-service
spec:
  type: ClusterIP
  clusterIP: None
  selector:
    app: webui
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

> ClusterIP가 None으로 명시하면 headless Service

* 실습

* 



















