Controller
================

* Controller 

> Pod의 개수를 보장

```
// nginx 웹서버 3개 실행
$ kubectl create deployment webui --image=nginx --replicas=3
```


1. ReplicationController
2. ReplicaSet
3. Deployment
4. DaemodSet
5. StatefunSet
6. Job
7. CronJob


## ReplicationController

> 요구하는 Pod의 개수를 보장하며 Pod 집합의 실행을 항상 안정적으로 유지하는 것을 목표

> 요구하는 Pod의 개수가 부족하면 template를 이용해 Pod 추가

> 요구하는 Pod수 보다 많으면 최근에 생성된 Pod를 삭제

* 기본구성

1. selector

2. replicas

3. template

> yaml file 형태
```
apiVersion: v1
kind: ReplicationController
metadata:
  name: [RC name]
spec:
  replicas: [배포 개수]
  selector:
    key: value
  template:
    [컨테이너 템플릿]

```

> ReplicationController-definition
```
apiVersion: v1
kind: ReplicationController
metadata:
  name: rc-nginx
spec:
  replicas: 3
  selector:
    app: webui
  template:
    metadata:
      name: nginx-pod
      labels:
        app: webui
      spec:
        containers:
        - name: nginx-controller
          image: nginx:1.14
```

* rc-nginx.yaml 실행
```
$ kubectl create -f rc-nginx.yaml

```

* replicationController 확인

```
$ kubectl get replicationcontrollers
```

* replicationController 조회

```
$ kubectl describe rc rc-nginx
```

> labels를 확인해서 key: value를 확인한다. 

> replicationcontroller에서 같은 labels를 사용하는 container를 제거한다.

* replicationController 실행 중 edit(replicas 개수)

```
$ kubectl edit rc rc-nginx
```

> vim에서 바로 수정가능

* scale out, scale down

```
$ kubectl scale rc rc-nginx --replicas=2
```

* 서버 엔진 버전 수정

```
$ kubectl edit rc rc-nginx
// vim을 통해 replication Controller 데이터 수정

$ kubectl delete pod [replicas name]
// pod가 삭제후 정해진 replicas의 수를 유지하기 위해 다시 시작
// 업데이트된 정보로 pod 생성

```

## ReplicaSet

> ReplicationController와 같은 역할을 하는 컨트롤러

> ReplcationController 보다 풍부한 Selector

```
selector:
  matchLabels:
    component: redis
  matchExpression:
    - {key: tier, operator: In, values: [cache]}
    - {key: environment, operator: NotIn, values: [dev]}
```

* MatchExpression 연산자

  + In: key와 value를 지정하여 key, value가 일치하는 Pod만 연결
  + NotIn: key는 일치하고 value는 일치하지 않는 Pod에 연결
  + Exists: key에 맞는 label의 Pod를 연결
  + DoesNotExits: key와 다른 label의 pod를 연결


* ReplicaSet(rs-nginx.yaml)

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rc-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webui
    matchExpressions:
      - {key: version, operator: In, values: ["1.14"]}
  template:
    metadata:
      name: nginx-pod
      labels:
        app: webui
    spec:
      containers:
      - name: nginx-controller
        image: nginx:1.14
```


* scale

```
kubectl scale rs rs-nginx --replicas=2
```

* Controller만 지우기 


```
$ kubectl delete rs rs-nginx --cascade=false
// controller만 지우고 controller에 의해 생성된 pod는 유지
```

* 실습 

1. ReplicaSet을 사용하는 rs-lab.yaml 파일을 생성하고 동작

- labes(name:apache, app:main, rel:stable)를 가지는 httpd:2.2 버전의 pod 2개 운영
  - rs name: rs-mainui
  - container: httpd:2.2
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rs-mainui
spec:
  replicas: 2
  selector:
    matchLabels:
     app: main
  template:
    metadata:
      name: apache
      labels:
        app: main
        rel: stable
    spec:
      containers:
      - image: httpd:2.2
        name: httpd
```

## Deployment

> ReplicaSet을 제어하는 부모 역할

> ReplicaSet을 컨트롤해서 Pod 수 조절

> Rolling update & Rolling Back(서비스 중단 없이 application update, Back)

* Deployment definition

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webui
  template:
    metadata:
      name: nginx-pod
      labels:
        app:webui
    spec:
      containers:
      - name: nginx-container
        image: nginx:1.14
```


* pod name

> deployment - controller - pod

> deploy-nginx-6d4c4cc4b8-kpglg

* Rolling Update

```
$ kubectl set image deployment <deploy name> <container name>=<new_version_image>

$ kubectl set image deployment deploy-nginx nginx-container=nginx:1.15 --record
```

* RollBack

```
// rollout history -> 이전 업데이트 기록 출력
$ kubectl rollout history deployment <deploy_name>
$ kubectl rollout undo deploy <deploy_name>
```

* 실습

+ deploy-exam1.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webui
  template:
    metadata:
      labels:
        app: webui
    spec:
      containers:
      - image: nginx:1.14
        name: web
        ports: 
        - containerPort: 80
```

+ rolling update

```
// update 기록 저장
$ kubectl create -f deploy-exam1.yaml --record

// update
$ kubectl set image deployment app-deploy web=nginx:1.15 --record

// update 상태 출력
$ kubectl rollout status deployment app-deploy

// 업데이트 일시정지
$ kubectl rollout pause deployment app-deploy

// 업데이트 재시작
$ kubectl rollout resume deployment app-deploy

// update 기록 출력
$ kubectl rollout history deployment app-deploy

// 업데이트 롤백(바로 전단계)
$ kubectl rollout undo deployment app-deploy


// 업데이트 롤백(revision 선택)
$ kubectl rollout undo deployment app-deploy --to-revision=3

```

* yaml file을 수정해서 rolling update

```
# dep-lab.yaml
apiVersion: apps/v1
kind: Deployment
metadata: 
  name: dep-ui
  annotations:
    kubernetes.io/change-cause: version 2.2
spec:
  progressDeadlineSeconds: 600
  revisionHistoryLimit: 10
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  replicas: 2
  selector:
    matchLabels:
      app: main
  template:
    metadata:
      labels:
        name: apache
        app: main
        rel: stable
    spec:
      containers:
      - name: web
        image: httpd:2.2
        ports:
        - containerPort: 80
```

```
//create가 아니라 apply 명령어로 실행
$ kubectl apply -f dep-lab.yaml

// vim 또는 파일에 접근해서 image 버전 수정

kubernetes.io/change-cause: version 2.2 -> kubernetes.io/change-cause: version 2.4

template의 image 버전 수정 image: httpd:2.2 -> image: httpd:2.4

$ kubectl apply -f dep-lab.yaml

// history 확인
$ kubectl rollout history 
```


























