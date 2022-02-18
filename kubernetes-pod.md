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











