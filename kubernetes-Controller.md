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
    - {key: environment, operator: NotIn, value: [dev]}
```

* MatchExpression 연산자

  + In: key와 value를 지정하여 key, value가 일치하는 Pod만 연결
  + NotIn: key는 일치하고 value는 일치하지 않는 Pod에 연결
  + Exists: key에 맞는 label의 Pod를 연결
  + DoesNotExits: key와 다른 label의 pod를 연결


* ReplicaSet

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
      - {key: version, operator: In, value: ["1.14]}
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





















