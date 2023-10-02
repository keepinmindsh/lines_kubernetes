# Section 4 - 레이블과 셀렉터, 네임스페이스에 대해서

## Cheat Sheet 

- namespace 

```shell 
# 실행하는 모든 kubectl command 명령어가 실행되는 NameSpace 지정하는 방법 
$ kubectl config set-context --current --namespace=<insert-namespace-name-here>
# Validate it
$ kubectl config view --minify | grep namespace:
```

```shell
kubectl create namespace my-namespace
```

## Basic Sample 

```yaml
apiVersion: v1 
kind: Service 
metadata: 
  name: endpoint1 
spec: 
  selector: 
    svc: endpoint 
  ports: 
    port: 8080
```


```yaml 
apiVersion: v1 
kind: Pod 
metadata: 
  name: pod7 
  labels: 
    svc: endpoint
spec: 
  containers:
    - name: container 
      image: kubetm/app 
```

## 레이블과 셀렉터를 이용한 파드 스케쥴링 제한

쿠버네티스는 모든 노드를 하나의 대규모 배포 플랫폼으로 노출하기 때문에, 파드가 어느 노드에 스케쥴링 됐느냐는 중요하지 않다. 각 파드는 요청한 만큼의
정확한 컴퓨팅 리소스를 할당 받는다.

### 하지만...

파드를 스케쥴링할 위치를 결정할 때 약간이라도 영향을 미치고 싶은 상황이 있다. 예를 들어 하드웨어 인프라가 동일하지 않는 경우를 들 수 있다. 워커 노드 일부는
HDD를 가지고 있고 나머지에는 SSD를 가지고 있는 경우, 특정 파드를 한 그룹웨 나머지 파드는 다른 그룹에 스케쥴링 되도록 할 수 있다.

### 워커노드 분류에 레이블 사용

노드를 포함한 모든 쿠버네티스 오브젝트에 레이블을 부탁할 수 있다. 일반적으로 ops 팀은 새 노드를 클러스터에 추가할 때, 노드가 제공하는 하드웨어나 파드를 스케줄링할 때
유용하게 사용할 수 있는 기타 사항을 레이블로 지정해 노드를 분류한다.

```shell
$ k get nodes
 
NAME                                           STATUS   ROLES    AGE     VERSION
gke-lines-cluster-default-pool-0f0b3237-ck61   Ready    <none>   7d22h   v1.24.9-gke.3200
gke-lines-cluster-default-pool-0f0b3237-l1ie   Ready    <none>   7d21h   v1.24.9-gke.3200
gke-lines-cluster-default-pool-0f0b3237-sfj8   Ready    <none>   7d22h   v1.24.9-gke.3200

$ k label node gke-lines-cluster-default-pool-0f0b3237-ck61 test=true 
node/gke-lines-cluster-default-pool-0f0b3237-ck61 labeled

$ k get node -l test=true 
NAME                                           STATUS   ROLES    AGE     VERSION
gke-lines-cluster-default-pool-0f0b3237-ck61   Ready    <none>   7d22h   v1.24.9-gke.3200

$ k get nodes -L test 
NAME                                           STATUS   ROLES    AGE     VERSION            TEST
gke-lines-cluster-default-pool-0f0b3237-ck61   Ready    <none>   7d22h   v1.24.9-gke.3200   true
gke-lines-cluster-default-pool-0f0b3237-l1ie   Ready    <none>   7d22h   v1.24.9-gke.3200
gke-lines-cluster-default-pool-0f0b3237-sfj8   Ready    <none>   7d22h   v1.24.9-gke.3200
```

#### NodeSelector를 통한 특정 Label의 Pod 배포

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: HelloWorld
  labels:
    name: HelloWorld
spec:
  nodeSelector:
    test: "true"
  containers:
  - name: HelloWorld
    image: gcr.io/google-containers/busybox
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
    ports:
      - containerPort: 8080
```

## 파드에 어노테이션 달기

파드 및 다른 오브젝트는 레이블 이외에 어노테이션을 가질 수 있다. 어노테이션은 키-값 쌍으로 레이블과 비슷하지만 식별 정보를 갖지 않는다. 레이블은 오브젝트를
묶는데 사용할 수 있지만, 어노테이션는 그럴 수 없다.  
어노테이션은 많은 정보를 보유할 수 있다. 이는 주로 도구들에서 사용된다. 특정 어노테이션은 쿠버네티스에 의해 자동으로 오브젝트에 추가되지만, 나머지 어노테이션은 사용자에 의해 수동으로 추가된다.  
어노테이션은 쿠버네티스에 새로운 기능을 추가할 때 흔히 사용된다.

```shell
$ k get deployment lines-admin-nextjs-deployment -o yaml
```

```yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"name":"lines-admin-nextjs-deployment","namespace":"default"},"spec":{"replicas":2,"selector":{"matchLabels":{"app":"lines-admin-nextjs"}},"template":{"metadata":{"labels":{"app":"lines-admin-nextjs"}},"spec":{"containers":[{"image":"gcr.io/lines-infra/lines_admin_front:v0.1.0","name":"lines-admin-nextjs","ports":[{"containerPort":3000}],"resources":{"limits":{"cpu":"1024m","memory":"1024Mi"},"requests":{"cpu":"100m","memory":"32Mi"}}}]}}}}
  creationTimestamp: "2023-01-12T13:11:58Z"
  generation: 3
  name: lines-admin-nextjs-deployment
  namespace: default
  resourceVersion: "54862222"
  uid: a8dcd5bf-0aed-4175-8fd3-c68c456034a4
spec:
  progressDeadlineSeconds: 600
```

### 실제로 annotations 추가하기

```shell 
$ k annotate deployment lines-admin-nextjs-deployment lines/admin-annotation="dream come true"
```

```shell
$ k describe deployment lines-admin-nextjs-deployment 

Name:                   lines-admin-nextjs-deployment
Namespace:              default
CreationTimestamp:      Thu, 12 Jan 2023 22:11:58 +0900
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
                        lines/admin-annotation: dream come true
```

## 네임스페이스를 사용한 리소스 그룹화

쿠버네티스 네임스페이스는 오브젝트 이름의 범위를 제공한다. 모든 리소스를 하나의 단일 네임 스페이스에 두는 대신에 여러 네임스페이스로 분할 할 수 있으며, 이렇게
분리된 네임스페이스는 같은 리소스 이름을 다른 네임스페이스에 걸쳐 여러번 사용할 수 있게 해준다.

### 필요한 부분은

여러 네임스페이스를 사용하면 많은 구성 요소를 가진 복잡한 시스템을 좀 더 작은 개별 그룹으로 분리할 수 있다. 또한 멀티 테넌스 환경 처럼 리소스를 분리하는데 사용된다.  
네임스페이스를 리소스를 격리하는 것 외에도 특정 사용자가 지정된 리소스에 접근할 수 있도록 허용하고, 개발 사용자가 사용할 수 있는 컴퓨팅 리소스를 제한하는 데에도 사용된다.

- 네임스페이스 조회 하고 Object 살펴보기

```shell
$ k get ns 
NAME                         STATUS   AGE
argocd                       Active   3d23h
default                      Active   91d
kube-node-lease              Active   91d
kube-public                  Active   91d
kube-system                  Active   91d
tekton-pipelines             Active   29d
tekton-pipelines-resolvers   Active   29d
```

```shell
$ k get configmap -n tekton-pipelines 
config-artifact-bucket     0      27d
config-artifact-pvc        0      27d
config-defaults            1      27d
config-leader-election     1      27d
config-logging             3      27d
config-observability       1      27d
config-registry-cert       0      27d
config-trusted-resources   1      27d
feature-flags              12     27d
kube-root-ca.crt           1      27d
pipelines-info             1      27d
```

- namespace를 yaml로 만들어보기 / 확인하기 / 지우기

```yaml
apiVersion: v1 
kind: Namespace 
metadata:
  name: custom-namespace
```

```shell
$ k create -f /Users/lines/sources/02_linesgits/lines_kubernetes/007_kuberntes_in_action/p146_create_namespaces/custom_namespace.yaml
namespace/custom-namespace created

$ k get namespaces 
NAME                         STATUS   AGE
argocd                       Active   4d
custom-namespace             Active   78s
default                      Active   91d
kube-node-lease              Active   91d
kube-public                  Active   91d
kube-system                  Active   91d
tekton-pipelines             Active   29d
tekton-pipelines-resolvers   Active   29d

$ k delete namespace custom-namespace
namespace "custom-namespace" deleted
```

### 다른 네임 스페이스의 오브젝트 관리 및 격리

```shell
$ k create -f deployment.yaml -n custom-namespace 
``` 

네임스페이스를 사용하면 오브젝트를 별도 그룹으로 분리해 특정한 네임스페이스 안에 속한 리소스를 대상으로 작업할 수 있게 해주지만,
실행 중인 오브젝트에 대한 격리는 제공하지 않는다.

## 파드 중지와 제거

```shell 
$ k delete po lines-sample
```

### Label Selector를 이용한 파드 삭제

```shell
$ k delete po -l create_method=manual

$ k delete po -l rel=canary  
```

### Namespace 전체 삭제

명령을 사용해 네임스페이스 전체를 삭제할 수 있다.

```shell
$ k delete ns custom-namespace 
```

### 네임스페이스를 유지하면서 네임스페이스 안에 있는 모든 파드 삭제

```shell
$ k get pods 
```

아래와 같은 삭제 방식은 Replication Controller가 남아 있을 경우 다시 파드가 생성된다.

```shell
$ k delete po --all 
```

namespace 내에서 존재하는 모든 오브젝트를 삭제하는 방식은 아래의 명령어를 활용할 수 있다.

```shell
k delete all -all 
```

> 해당 명령은 Kubernetes 서비스도 삭제하지만 잠시 후에 자동으로 다시 생성된다.

## 파드를 안정적으로 유지하기

쿠버네티스를 사용하면 얻을 수 있는 주요 이점은 쿠버네티스에 컨테이너 목록을 제공하면 해당 컨테이너를 클러스터 어딘가에서 계속 실행되도록 할 수 있다는 것이다.

### 프로세스를 요약하여 정리해보면

파드가 노드에 스케쥴링 되는 즉시, 해당 노드의 Kubelet은 파드의 컨테이너를 실행하고 파드가 존재하는 한 컨테이너가 계속 실행되도록 할 것이다.
컨테이너의 주 프로세스에 크래쉬가 발생하면 Kubelet이 컨테이너를 다시 시작한다. 만약 여러분의 애플리케이션에 버그가 있어 가끔식 크래시가 발생하는 경우
쿠버네티스가 애플리케이션을 자동으로 다시 시작하므로, 애플리케이션에서 특별한 작업을 하지 않더라도 쿠버네티스에서 애플리케이션을 다시 시행하는 것만으로도
자동으로 치유할 수 있는 능력이 주어진다. 

## 쿠버네티스 공식 문서를 통해 이해해야할 것 

> [Namespace와 DNS](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/#namespaces-and-dns)