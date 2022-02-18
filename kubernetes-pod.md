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

> 특정 container를 실행하기 전에 init container를 실행하고 실패 시 main container 생성 x


















