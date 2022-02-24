Ingress
==========

1. 인그레스의 이해
2. 인그레스 컨트롤러 설치
3. 실습 : 인그레스를 이용해 웹서비스 운영



## 인그레스의 이해

* HTTP나 HTTPS를 통해 클러스터 내부의 서비스를 외부로 노출

* 기능
  + Service에 외부 URL 제공
  + 트래픽을 로드밸런싱
  + SSL 인증서 처리
  + Virtual hosting을 지정
  
* kubernetes Ingress 동작방식

1. 생성한 노드에 각각의 서비스를 제공하는 Pod를 생성

> 이때 ClusterIP(단일진입점) 서비스를 사용하기 때문에 외부에서 접속 불가

2. Ingress Controller를 통해 각각의 ClusterIP에 연결해주는 Controller 생성

3. 외부에서 요청한 URL에 맞는 Service에 트래픽 요청 -> Service의 key:value에 맞는 Pod에 요청 전달

> 외부 클라이언트가 내부 클러스터에 접속할 수 있는 기능 제공 및 https 또한 제공

## Ingress 설치

```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.1/deploy/static/provider/aws/deploy.yaml
```


# Ingress를 이용한 웹서비스 운영

```

$ kubectl config set-context ingress-admin@kubernetes --cluster=kubernetes --user=kubernetes-admin --namespace ingress-nginx

$ kubectl config use-context ingress-admin@kubernetes

$ kubectl config set-context --current --namespace=ingress-nginx

```

* ingress rule

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: marvel-ingress
spec:
  rules:
  - http:
      paths:
      - path: /
        backend:
          serviceName: marvel-service
          servicePort: 80
      - path: /pay
        backend:
          serviceName: pay-service
          servicePort: 80
```

* yaml file 실행

```
$ kubectl create -f marvel-home.yaml -f pay.yaml

$ kubectl create -f ingress.yaml

```












