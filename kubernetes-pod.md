POD
=====


* Container

> 하나의 애플리케이션 

1. 컨테이너를 실행하는 K8s의 최소 api pod
2. Pod 동작, livenessProbe를 사용한 self-healing Pod
3. init Container, infra container(pause)
4. static pod, Pod에 resource 할당
5. 환경변수를 이용해 컨테이너에 데이터 전달하기, pod 구성 패턴의 종류

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

```
// create pod (CLI)

kubectl run web1 --image=nginx:1.14 --port=80

// 만든 pod를 yaml형태로 출력, json도 가능
//grep 명령어와 같이 사용하여 원하는 정보 출력 가능

kubectl get pods [objectname] -o yaml

kubectl get pods [objectname] -o yaml | grep -i "word"



// create pod with yaml file

kubectl create -f pod-nginx.yaml

// 2초마다 뒤의 명령어를 실행하여 모니터링
watch [command] ...

// 커맨드라인에서 웹 브라우저를 보여주는 명령어

curl [ip address]


```

## Multiple Container Pod

> 하나의 pod에 여러 컨테이너 실행

* yaml file
```
apiVersion: v1
kind: Pod
metadata:
  name: multipod
spec:
  containers:
  - name: nginx-container
    image: nginx:1.14
    ports:
    - containerPort: 80
  - name: centos-container
    image: centos:7
    command:
    - sleep
    - "10000"
```

## Pod 동작 flow


* Pod api 실행시

1. api의 형식이 맞는지 확인

2. etcd에서 Node에 대한 정보를 Controller가 확인

3. Controller에서 api 명령어를 실행하기 위해서 scheduler에 전달

> 이 때까지 Pending 상태

4. 어떤 Node에서 실행할지 scheduler가 정하고 Pod의 api를 선택한 Node에서 실행

> Running 상태


## 실습

```
// kubenetes pods 확인 
kubectl get pods -o wide --watch

// 실행중인 pods 전체 삭제
kubectl delete pod --all


// nginx image pod 실행
kubectl create -f pod-nginx.yaml

// 특정 pod 삭제
kubectl delete pod pod-nginx

// pod 수정
kubectl edit pod pod-nginx


```

* 컨테이너 image 'redis123'을 실행하는 pod 'redis'를 redis.yaml을 이용해 생성하시오 

```
kubectl run redis --image=redis123 --dry-run -o yaml > redis.yaml

kubectl create -f redis.yaml
```

> trouble shooting

```
kubetcl describe pod redis

// 정보확인
//image redis123은 없기 때문에 수정해서 사용
//또는 image 가져오기
```



## livenessProbe를 이용해 self-healing Pod

> kubelet으로 컨테이너 진단(건강검진)

```
// Pod definition

apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx-container
    image: nginx:1.14
    
// livenessProbe 

apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx-container
    image: nginx:1.14
    livenessProbe:
      httpGet:
        path: /
        port: 80
        
// http get method, 80포트로 건강검진(?)
```

* 검진 방식

1. httpGet

> 200값이 아니면(정상작동이 아니면) 오류, 컨테이너를 다시 시작

> web Servive를 사용한다면, 어떤 프레임워크나 언어를 사용하든지 상관없이 사용할 수 있다. 

2. tcpSocket

> 지정된 포트에 tcp 연결 시도, 연결되지 않으면 컨테이너 다시 시작

```
livenessProbe:
      tcpSocket:
        port: 8080
```

3. exec

> exec 명령을 전달하고 명령의 종료 코드가 0이 아니면 컨테이너를 다시 시작한다.

```
livenessProbe:
      exec:
        - ls
        - /data/file
```

* liveness probe 매개변수

1. periodSeconds: health check 반복 실행 시간(초)
2. initialDelaySeconds: Pod 실행 후 delay할 시간(초)
3. timeoutSeconds: health check 후 응답을 기다리는 시간(초)
4. failureThreshold: threshold 횟수 이상 실패 시 컨테이너를 죽이고 새로 컨테이너를 생성



## 실습

```
// yaml file

apiVersion: v1
kind: Pod
metadata:
  name: liveness-exam
spec:
  containers:
  - name: busybox-container
    image: busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - ls
        - /tmp/healthy
      initialDelaySeconds: 10
      failureThreshold: 2
      periodSeconds: 5
      successThreshold: 1
      timeoutSeconds: 1
```


## init container를 적용한  pod

> 특정 container를 실행하기 전에 init container를 실행하고 실패 시 main container 실행 x

```
// init-container.yaml

apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
```

> init Container가 정상 수행시 main container가 실행되는 것을 확인 

* https://kubernetes.io/ko/docs/concepts/workloads/pods/init-containers/

## infra Container(pause) 이해하기

> Pod의 환경을 만들어주는 Container

> Pod 생성시 pause Container가 자동으로 생성되고 삭제될 때 같이 삭제 

> Pod에 대한 IP, HostName을 관리해주는 Container


* 실습
```
// Pod 생성 후 Pod가 생성된 Node로 이동

$ kubectl run webserver --image=nginx:1.14 

$ ssh [Node Name]

$ docker ps

// ~~~ 결과 출력
// nginx 컨테이너가 실행되는 동시에 pause도 같이 생성
```

## static Pod

> api에 요청을 보내지 않고 Pod 생성

> kubelet demon에 의해 동작되는 Pod

> /etc/kubernetes/mainfests/ 디렉토리에 k8s yaml 파일을 저장시 적용됨

* static pod 디렉토리 구성
```kuber
$ vi /var/lib/kubelet/config.yaml

...
staticPodPath: /etc/kubernetes/manifests
// 디렉토리를 직접 수정해서 pod 실행 
```

* 디렉토리 수정시 kubelet 데몬 재실행

```
systemctl restart kubelet
```

## Pod에 리소스(CPU, Memory) 할당하기

> 각 Node에 할당된 CPU, Memory를 Pod에 할당(resource를 제한)

> Node에 할당된 모든 resource를 사용할 경우 서버가 죽는 경우가 생김

* Resoursce Requests

> 파드를 실행하기 위한 최소 리소스 양 요청

* Resource Limits

> 파드가 사용할 수 있는 최대 리소스 양을 제한

> Memory limit을 초과해서 사용하는 파드는 종료되며 다시 스케줄링 된다.

* 실습

```
// pod-nginx-resources.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-resource
spec:
  containers:
  - name: nginx-container
    image: nginx:1.14
    ports:
    - containerPort: 80
      protocol: TCP
    resources:
      requests:
        cpu: 200m
        memory: 250Mi
      limits:
      cpu: 1
      memory: 500Mi
      
// 1 Mb = 1000Kb
//   1 Mi = 1024 Kib

//cpu -> m(milli core)

```

> limit과 request를 적절히 배분해서 사용하는 것이 중요

> pending 상태(scheduler에 들어간 상태)에서 running으로 진행이 되지 않을 수 있음




## Pod의 환경변수 설정하기 

> Pod내의 컨테이너가 실행될 때 필요로 하는 변수

> 컨테이너 제작시 미리 정의

* ex)
```
ENV NGINX_VERSION 1.19.2
ENV NJS_VERSION 0.4.3
```

> Pod 실행 시 미리 정의된 컨테이너 환경변수를 변경할 수 있다.


```
# pod-nginx-env.yaml

apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-env
spec:
  containers:
  - name: nginx-container
    image: nginx:1.14
    ports:
    - containerPort: 80
      protocol:  TCP
    env:
    - name: MYVAR
      value: "testvalue"    

# 생성된 pod에 접속하기
$ kubectl exec nginx-pod-env -it -- /bin/bash

# 환경변수 출력
$ env
```

## Pod 구성 패턴의 종류

* Pod 실행 패턴

1. multi-container Pod

 + Sidecar Pod
 
 > 컨테이너 2가지가 동시에 수행되어야 정상 수행되는 Pod
 
 + Adapter
 
 > 외부의 모니터링 정보를 가져오는 container와 앱을 구동하는 container로 구성
 
 + Ambassador

 > 웹서버가 받은 데이터를 여러 저장소에 분산하여 저장하는 역할을 담당하는 컨테이너를 가짐


* Pod 실습

1. static pod 생성

```
#노드 접속
#/etc/kubernetes/mainfests에서 yaml파일 생성
#mydb-node01.yaml

apiVersion: v1
kind: Pod
metadata:
  name: mydb-node01
spec:
  containers:
  - image: redis
    name: mydb
```

2. resource 할당, 환경변수 설정, namespace 지정

```
# product namespace 생성
$ kubectl create namespace product

#yaml 파일 생성

apiVersion: v1
kind: Pod
metadata:
  name: myweb
spec:
  containers:
  - image: nginx:1.14
    name: myweb
    env:
      - name: DB
        value: mydb
    resources:
      requests:
        memory: 500Mi
        cpu: 200m
      limits:
        memory: 1Gi
        cpu: 1
```




























