쿠버네티스 아키텍처
======================== 

## namespace 

* nginx.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: --
spec:
  containers:
  - name: nginx
    image: nginx:1.7.9
    ports:
    - containerPort: 80
    - containerPort: 443
```

* kubectl create namespace [objectName]

* kubectl get namespace

* kubectl create -f [name.yaml file]

> 파일을 불러와 오브젝트 생성

* CLI 방식

  + kubectl create namespace blue

  + kubectl create namespace orange --dry-run -o yaml > orange-ns.yaml

 
 * kubectl create -f __.yaml -n [namespace]
 
 > yaml 파일을 지정한 namespace 안에서 실행



* namespace 바꾸기

* kubectl config set-context blue@kubernetes --cluster=kubernetes --user=kubernestes-admin --namespace=blue

* kubectl config current-context

* kubectl config use-context blue@kubernetes


![image](https://user-images.githubusercontent.com/94096054/154427801-d53ec187-a239-4add-a8d8-cccdcea183b5.png)

> 쿠버네티스의 구조는 위 그림과 같이 워커 노드에 각 pod들이 배포되는 구조

> pod이외에도 여러 리소스를 사용하는데 비슷한 이름의 오브젝트를 생성하게 된다면 관리와 사용 측면에 어려움을 겪음

> 이를 해결하기 위해 namespace를 사용하여 쿠버네티스 내의 논리적인 분리를 하여 사용


## yaml 템플릿

> 사람이 쉽게 읽을 수 있는 데이터 직렬화 양식

> Python처럼 들여쓰기로 데이터 계층 표기

> tab이 아닌 space bar 사용

> Key : value를 설정 

> '-' 사용하여 배열처럼 사용

## API 버전 

> Object 정의 시 apiVersion이 필요

* API Object의 종류 및 버전

  + Deployment : apps/v1

  + Pod : v1

  + ReplicaSet : apps/v1

  + ReplicationController : v1

  + Service : v1

  + PersistentVolume : v1





