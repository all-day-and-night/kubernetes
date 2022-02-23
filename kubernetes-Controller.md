Controller
================

* Controller 

> Pod의 개수를 보장

```
// nginx 웹서버 3개 실행
$ kubectl create deployment webui --image=nginx --replicas=3
```


1. ReplicationController
> pod 개수 보장 컨트롤러

2. ReplicaSet
> Replication Controller + 풍부한 Label 지원 

3. Deployment
> ReplicaSet 제어(Rolling update/ Roll back)

4. DaemodSet
> Node 하나당 1개씩 pod 실행 보장

5. StatefunSet
> Pod의 이름 유지

6. Job
> Pod의 정상적인 종료를 관리

7. CronJob
> Job을 스케줄링하여 예약 사용 지원


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



## DaemonSet

> 전체 노드에서 Pod가 한 개씩 실행되도록 보장

> 로그 수입기, 모니터링 에이전트와 같은 프로그램 실행시 적용

> replica의 개수를 지정하지 않아도 된다.(node의 개수만큼 실행되기 때문)

> 추가로 노드를 생성하더라도 자동으로 노드에 pod가 실행된다.

> Rolling update 가능

* DaemonSet definition

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: daemonset-nginx
spec:
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


## StatefulSet

> Pod의 상태를 유지해주는 컨트롤러

> Pod의 이름, Pod의 볼륨(Storage)

* StatefulSet으로 nginx 3개 실행

> 만약 pod하나가 문제가 생긴다면, Pod를 종료하고 이후에 종료된 Pod의 이름을 가진 Pod를 실행한다.

> 이 때, Pod가 생성되는 Node의 위치는 달라질 수 있다.

* StatefulSet definition

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: sf-nginx
spec:
  replicas: 3
  serviceName: sf-nginx-service
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

> ServiceName 명시 

* statefulset-exam.yaml

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: sf-nginx
spec:
  replicas: 3
  serviceName: sf-service
#  podManagementPolicy: OrderedReady
  podManagementPolicy: Parallel
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

* scale

```
$ kubectl scale statefulset sf-nginx --replicas=2
```

* rolling update

```
$ kubectl edit statefulset sf-nginx 
```

* rollback

```
kubectl rollout undo statefulset sf-nginx
```

* rollback (revision 선택)

```
kubectl rollout undo statefulset sf-nginx --to-revision=[revision number]
```


## Job Controller

* 쿠버네티스는 pod를 running 중인 상태로 유지
* Batch 처리하는 Pod는 작업이 완료되면 종료됨
* Batch 처리에 적합한 컨트롤러로 Pod의 성공적인 완료를 보장
  + 비정상 종료시 다시 실행
  + 정상 종료시 완료

* exam

```
// pod 생성 후 5초 뒤에 제거
$ kubectl run testpod --image=centos:7 --command sleep 5

// pod의 상태를 확인해보면 pod는 생성 후 5초 뒤에 제거되고
// 비정상적으로 종료된 Pod를 kubernetes가 다시 생성한다.
// ...반복
```

* Job Controller definition

```
apiVersion: apps/v1
kind: Job
metadata:
  name: job-example
spec:
  template:
    spec:
      containers:
      - name: centos-container
        image: centos:7
        command: ["bash"]
        args:
        - "-c"
        - "echo 'hello World'; sleep 50; echo 'Bye'"
      restartPolicy: Never
```




* job-exam.yaml

```
apiVersion: batch/v1
kind: Job
metadata:
  name: job-example
spec:
#  실행해야 할 job의 수가 몇개인지 지정, replicas와 유사(하지만 palrallelism을 명시하지 않으면 동시에 running되지 않음)
#  completions: 5
#  병렬성, 동시 running 되는 pod 수
#  parallelism: 2
#  지정 시간 내에 Job을 완료, 그 시간 이전에 종료되지 않으면 강제로 completed로 상태 변환
#  activeDeadlineSeconds: 15
  template:
    spec:
      containers:
      - name: centos-container
        image: centos:7
        command: ["bash"]
        args:
        - "-c"
        - "echo 'hello World'; sleep 50; echo 'Bye'"
      restartPolicy: Never
```

> job에서 지정한 업무를 완료하고 정상 종료된다면 pod는 사라지지 않고 completed 상태로 바뀐다.

## CronJob

> Jon 컨트롤러로 실행할 Application Pod를 주기적으로 반복해서 실행

> Linux의 cronjob의 스케줄링 기능을 Job Controller에 추가한 api

> 다음과 같은 반복해서 실행하는 Job을 운영할 때 사용

  * Data Backup
  * Send email
  * Cleaning tasks

* CronJon schedule: "0 9 1 * *" , 매월 1일 9시 정각에 job 실행해줘
  + Minutes(from 0 to 59)
  + Hours(from 0 to 23)
  + Day of the month(from 1 to 31)
  + Month(from 1 to 12)
  + Day of the Week(from 0(sun) to 6(sat))

> 매주 일요일 새벽 3시에 job 실행

> "0 3 * * 0"

> 주중에 새벽 3시에 job 실행

> "0 3 * * 1-5"

> 5분마다

> "*/5 * * * *"

> 매일 5분이랑 10분

> "5,10 * * * *"

* CronJob definition

```
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: cronjob-definition
spec:
  schedule: "0 3 1 * *"
  jobTemplate:
    spec:
      containers:
      - name: hello
        image: busybox
        args:
        - /bin/sh
        - -c
        - date; echo hello
      restartPolicy: Never

```

* cronjob-exam.yaml
```
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cronjob-exam
spec:
  schedule: "* * * * *"
#  startingDeadlineSeconds: 500
#  Pod 2개 이상 실행 허용
#  concurrencyPolicy: Allow
#  허용 x
#  concurrencyPolicy: Forbid
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - echo Hello; sleep 10; echo Bye
          restartPolicy: Never
```









