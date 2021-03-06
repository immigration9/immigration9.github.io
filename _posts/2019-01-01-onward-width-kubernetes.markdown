---
layout: post
title: 'Kubernetes 101'
date: 2019-01-01 10:00:00 +09:00
categories: 'kubernetes'
published: true
---

## Kubernetes란?

무엇인가: 여러개의 장비 위에 여러개의 컨테이너를 돌리고 있는 시스템을 위해 사용
왜 사용하는가: 다른 이미지를 가지고 있는 여러개의 컨테이너를 돌려야할 때 사용

## 과정

Minikube로 설치

kubectl: 노드에 있는 컨테이너 관리 (k8s master와 커뮤니케이션)
minikube: VM 자체를 관리 (해당 VM에 싱글 노드 실행)

1. kubectl 설치: https://kubernetes.io/docs/tasks/tools/install-kubectl/
2. Virtualbox 설치: https://www.virtualbox.org/wiki/Downloads
3. minikube 설치: https://github.com/kubernetes/minikube/releases

- Minikube가 제공하는 Kubernetes 기능들
  DNS / NodePorts / ConfigMaps & Secrets / Dashboards / Container Runtime(Docker) / Container Network Interface 활성화 / Ingress

## 설치

MacOS 기준

1. kubectl: `brew install kubectl`
2. Virtualbox 설치
3. minikube: `brew cask install minikube`
4. minikube 시작: `minikube start`

- k8s master 상태 확인: `kubectl cluster-info`

## 목표

Get the multi-client image running on our local kubernetes cluster running as a container

## 실행

```shell
$ minikube start
Starting local Kubernetes cluster...
Running pre-create checks...
Creating machine...
Starting local Kubernetes cluster...

$ kubectl run hello-minikube --image=k8s.gcr.io/echoserver:1.10 --port=8080
deployment.apps/hello-minikube created
$ kubectl expose deployment hello-minikube --type=NodePort
service/hello-minikube exposed

# We have now launched an echoserver pod but we have to wait until the pod is up before curling/accessing it
# via the exposed service.
# To check whether the pod is up and running we can use the following:
$ kubectl get pod
NAME                              READY     STATUS              RESTARTS   AGE
hello-minikube-3383150820-vctvh   0/1       ContainerCreating   0          3s
# We can see that the pod is still being created from the ContainerCreating status
$ kubectl get pod
NAME                              READY     STATUS    RESTARTS   AGE
hello-minikube-3383150820-vctvh   1/1       Running   0          13s
# We can see that the pod is now Running and we will now be able to curl it:
$ curl $(minikube service hello-minikube --url)


Hostname: hello-minikube-7c77b68cff-8wdzq

Pod Information:
	-no pod information available-

Server values:
	server_version=nginx: 1.13.3 - lua: 10008

Request Information:
	client_address=172.17.0.1
	method=GET
	real path=/
	query=
	request_version=1.1
	request_scheme=http
	request_uri=http://192.168.99.100:8080/

Request Headers:
	accept=*/*
	host=192.168.99.100:30674
	user-agent=curl/7.47.0

Request Body:
	-no body in request-


$ kubectl delete services hello-minikube
service "hello-minikube" deleted
$ kubectl delete deployment hello-minikube
deployment.extensions "hello-minikube" deleted
$ minikube stop
Stopping local Kubernetes cluster...
Stopping "minikube"...
```

## Docker Compose와의 비교

- Docker Compose

1. 각각의 엔트리는 docker-compose로 하여금 이미지를 빌드하도록 할 수 있다.
2. service 안에 있던 nginx / worker / client와 같이 각각의 엔트리는 생성하고자 하는 컨테이너를 나타낸다.
3. 각각의 엔트리들은 네트워크 필요요소를 정의한다. (port mapping etc...)

- Kubernetes

1. 모든 이미지가 다 빌드 되어 있는 상태
2. 여러개의 config 파일이 있으며, 각각의 config 파일은 Object 하나당. (Object가 꼭 container는 아니다)
3. Networking은 모두 세팅을 별도로 해줘야 한다.

## Kubernetes로 시작하기

`mkdir simplek8s`

- Pod란: Container들의 그룹핑 (k8s에는 container만 만든다는 것이 없다. 최소 단위가 Pod)
- Pod에 담아야 하는 항목들: 굉장히 결합도가 높은 container들 끼리의 묶음. 정말 묶지 않으면 안될 정도

- Service란: k8s cluster의 네트워킹 설정

* ClusterIP / NodePort / LoadBalancer / Ingress
* NodePort란: Container를 외부에 노출 (개발용도로만 적합, 프로덕션에서는 되도록 사용하지 않음)

`client-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: client-pod
  labels:
    component: web
spec:
  containers:
    - name: client
      image: stephengrider/multi-client
      ports:
        - containerPort: 3000
```

`client-node-port.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: client-node-port
spec:
  type: NodePort
  ports:
    - port: 3050
      targetPort: 3000
      nodePort: 31515
  selector:
    component: web
```

- label matching이 되어야 하기 때문에, `labels` 항목과 `selector` 항목의 key value가 맞어야 한다.

NodePort Service의 구성

- port: `multi-client` pod를 필요로 하는 다른 pod
- targetPort: `multi-client` pod
- nodePort: 30000-32767 사이의 랜덤포트 (외부 머신에서 접속). 포트를 별도로 지정하지 않으면 랜덤으로 들어간다.

## kubectl을 이용하여 Pod 올리기

`kubectl apply -f (filename)`

- apply: cluster의 현재 설정을 변경
- -f (filename): 설정이 변경될 파일 명칭 지정

`kubectl get pods`: pod list
`kubectl get services`: service list

`kubectl get pods`의 결과물을 가지고 localhost:(nodeport)로 접속하고자 하면, 안된다.

`kube-proxy`를 통해서 해당 VM에 접근해야하기 때문이다.

minikube가 그 정보를 가지고 있기 때문에 `minikube ip`를 입력하여 ip주소를 가져온 뒤 사용하면 된다.
ex) 192.168.99.100:31515

`minikube ssh`: minikube 환경에 접속

Deployment file이 적용되면 master로 전송된다. (kube-apiserver).
master(kube-apiserver)는 deployment file을 읽은 뒤 작업을 수행한다.

## 용어 정리

- Kubernetes: 컨테이너화된 앱을 배포
- Nodes: Container들을 실행시키는 개별 머신 (혹은 VM)
- Masters: Node를 managegㅏ기 위한 프로그램을 갖추고 있는 개별 머신 (혹은 VM)
- Kubernetes는 이미지를 빌드하지 않음. 다른 곳에서 가져 옴
- Deploy를 하기 위해서는, master에 deploy 설정 파일을 전달한다.
- Master는 예상되는 상태에 부응하기 위해 끊임없이 동작한다.

## declarative vs imperative deployment

## 설정 변경의 기준

kubernetes 설정 파일의 경우, `name`과 `type` 항목을 기준으로 설정 파일을 업데이트 한다.
두 값들이 같을 경우, 다른 항목의 변경은 덮어쓰기 형식으로 적용된다. (image, toleration, activeDeadlineSeconds와 같은 지표들만 변경이 가능하다)

### 객체에 대한 상세 정보 가져오기

`kubectl describe (type: eg. pods) (name: client-pod)`

### 변경할 수 없는 Pod 설정을 변경하는 방법

- Deployment: 동일한 Pods들을 관리한다. 모두 동일한 설정값을 가지고 있으며, 정확한 숫자가 존재하도록 관리한다.

* Pod 대신에 사용할 수 있다. Pods는 일반적으로 프로덕션 환경에서 잘 사용되지 않는다.
* Pod들의 상태를 모니터링하고, 필요시 업데이트도 진행한다.

Pod Template을 기준으로 Pod를 생성한다.

## 기존 설정 파일 삭제

`kubectl delete -f (config file)`
상당히 명령적인 방식인데 (선언적인 방식과 반대인), 일단 삭제할 때는 이 방법 밖엔 없다.

## Deployment 가져오기

`kubectl get deployments`

- DESIRED: deployment 설정에서 설정한 replica 숫자
- CURRENT: 현재 돌아가는 Pod 수
- UP-TO-DATE: 설정 파일을 변경하면 일시적으로 기존의 Pod들이 out-of-date 상태가 된다. 그러면 숫자가 떨어졌다가, 설정이 반영되면 다시 올라온다.
- AVAILABLE: 잘 돌아가고 있는 Pod의 수

## Deployment에 대해

내용이 변경되면 기존의 Pod를 삭제하고 새로운걸 띄운다.

## Service는 왜 사용할까?

모든 Pod들은 다 개별적인 내부 IP를 가져간다. 내부 IP는 어떤 방식으로던 간에 변경될 수 있다.
Pod는 유기적이기 때문에 언제 사라질 수도, 교체될 수도 있다. 이렇게 되면 IP는 변경된다.

- 결론: EIP쓰는 이유와 같다. **Service는 Pod를 Selector를 기준으로 모니터링하고, 포트를 매핑한다.**

## Deploy시 image 버전 업데이트

단순히 latest는 자동으로 업데이트 되지 않는다. (kubernetes issue #33664 번 참조)
그렇기 때문에, kubectl 명령어를 통해 deployment가 새로운 이미지 버전을 사용하도록 강제한다.

1. docker build를 먼저 진행하자

```
docker build -t immigration9/multi-client:v2 ./
docker push immigration9/multi-client:v2
```

2. kubectl에 최신 이미지를 사용하도록 명령하자
   `kubectl set image deployment/client-deployment client=immigration9/multi-client:v2`
   kubectl set (변경하고자 하는 속성. eg. image) (객체 타입. eg. deployment) / (객체 이름) (컨테이너명)=(풀 이미지명)

이제 `kubectl get pods`를 하면 이미지가 업데이트 되며 Pod들이 재생성 된 것을 확인할 수 있다.
