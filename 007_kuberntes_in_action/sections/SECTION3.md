# Section 3 - 파드의 기본 

## Cheat Sheet 

```shell
kubectl create poddisruptionbudget my-pdb --selector=app=rails --min-available=1

kubectl create pdb my-pdb --selector=app=nginx --min-available=50%
```

## 파드에 대하여

파드 안에 있는 모든 컨테이너는 같은 노드에서 실행된다. 절대로 두 노드에 걸쳐 배포되지 않는다.

- 컨테이너는 단일 프로세스를 실행하는 것을 목적으로 설계했다.
    - 단일 컨테이너에서 관련 없는 다른 프로세스를 실행하는 경우 모든 프로세스를 실행하고 로그를 관리하는 것은 모두 사용자 책임이다. 일례로 개별 프로세스가 실패하는 경우 자동으로 재시작하는 메커니즘을 포함해야 한다.

- 여러 프로세스를 단일 구조로 묶지 않기 때문에, 컨테이너를 함께 묶고 하나의 단위로 관리할 수 있는 또 다른 상위 구조가 필요하다.

### 기본 파드의 정의 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

파드를 직접적으로 정의하기 보다는 주로 Deployment, Job, StatefulSet 내에서 정의되어 많이 사용된다. 

- Pods 는 하나의 단일 컨테이너로서 동작한다. 
- Pods 는 강하게 결합되어 동작해야하는 경우 다중 컨테이너를 하나의 파드 내에 구성하여 리소스를 공유할 수 있습니다.

#### Pod Template 으로서 Kubernetes 오브젝트 내에서 활용하기 

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: hello
spec:
  template:
    # This is the pod template
    spec:
      containers:
      - name: hello
        image: busybox:1.28
        command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']
      restartPolicy: OnFailure
    # The pod template ends here
```

### 같은 파드에서 컨테이너간 부분 격리

파드의 모든 컨테이너는 동일한 네트워크 네임스페이스와 UTS 네임스페이스 안에서 실행되기 때문에, 모든 컨테이너는 같은 호스트 이름과 네트워크 인터페이스를 공유한다.  
파드의 모든 컨테이너는 동일한 IPC 네임스페이스 아래에서 실행돼 IPC를 통해 서로 통신할 수 있다.

### 컨테이너가 동일한 IP와 포트 공간을 공유하는 방법

파드 안의 컨테이너가 동일한 네트워크 네임스페이스에서 실행되기 때문에, 동일한 IP 주소와 포트 공간을 공유한다는 것이다.  
이는 동일한 파드 컨테이너에서 실행중인 프로세스가 같은 포트 번호를 사용하지 않도록 주의해야함을 의미한다.

### 파드 간 플랫 네트워크 소개

쿠버네티스의 모든 파드는 하나의 플랫한 공유 네트워크 주소 공간에 상주하므로 모든 파드는 다른 파드의 IP 주소를 사용해 접근하는 것이 가능하다.
둘 사이에서는 어떠한 NAT 도 존재하지 않는다. 두 파드가 서로 네트워크 패킷을 보내면, 상대방의 실제 IP 주소를 패킷 안에 있는 출발지 IP 주소에서 찾을 수 있다.

- 동일한 network namespace 내의 하나의 파드 내의 컨테이너는 IP, Port를 모두 공유한다. 
  - Pod 내의 Container에서의 호출은 Localhost를 이용하여 호출이 가능하다. 

### 파드에서 컨테이너의 적절한 구성

#### 파드에서 여러 컨테이너를 사용하는 경우

- 컨테이너를 함께 실행해야 하는가, 혹슨 서로 다른 호스트에서 실행할 수 있는가?
- 여러 컨테이너가 모여 하나의 구성 요소를 나타내는가, 혹은 개별적인 구성요소 인가?
- 컨테이너가 함께, 혹은 개별적으로 스케일링돼야 하는가?

**컨테이너는 여러 프로세스를 실행하지 말아야 한다. 파드는 동일한 머신에서 실행할 필요가 없다면 여러 컨테이너를 포함하지 말아야 한다.**

## Pod's LifeCycle 

- Pod's Phase 
  - Pending 
  - Running 
  - Succeeded 
  - Failed 
  - UnKown 

- Container State 
  - Waiting
  - Running 
  - Terminated

## Pod's Restart Policy 

- restartPolicy 
  - Always ( default value )
  - OnFailure 
  - Never 

## Pod's Conditions 

- PodScheduled
- PodHasNetwork 
- ContainerReady 
- Ready 

## Sample of Pod Status 

```yaml 
kind: Pod
...
spec:
  readinessGates:
    - conditionType: "www.example.com/feature-1"
status:
  conditions:
    - type: Ready                              # a built in PodCondition
      status: "False"
      lastProbeTime: null
      lastTransitionTime: 2018-01-01T00:00:00Z
    - type: "www.example.com/feature-1"        # an extra PodCondition
      status: "False"
      lastProbeTime: null
      lastTransitionTime: 2018-01-01T00:00:00Z
  containerStatuses:
    - containerID: docker://abcd...
      ready: true
...
```

## YAML 또는 JSON 디스크립터로 파드 생성

### [API References](https://kubernetes.io/docs/reference/)

### YAML 디스크립터 이해하기

- Metadata : 이름, 네임스페이스, 레이블 및 파드에 관한 기타 정보 포함
- Spec : 파드 컨테이너, 볼륨, 기타 데이터 등 파드 자체에 관한 실제 명세를 가진다.
- Status : 파드 상태, 각 컨테이너 설명과 상태, 파드 내부 IP, 기타 기본 정보 등 현재 실행 중인 파드에 관한 현재 정보를 포함한다.

```shell
k get deployment lines-admin-nextjs-deployment -o yaml
```

```yaml
apiVersion: apps/v1  # YAML 디스크립터에서 사용한 쿠버네티스 API 버전
kind: Deployment     # 쿠버네티스 오브젝트/리소스 유형 
metadata:            # 파드 메타 데이터 ( 이름, 레이블, 어노테이션 등 ) 
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
spec:               # 정의 / 내용 
  progressDeadlineSeconds: 600
  replicas: 2
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: lines-admin-nextjs
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: lines-admin-nextjs
    spec:
      containers:
      - image: gcr.io/lines-infra/lines_admin_front:v0.1.0
        imagePullPolicy: IfNotPresent
        name: lines-admin-nextjs
        ports:
        - containerPort: 3000
          protocol: TCP
        resources:
          limits:
            cpu: 1024m
            memory: 1Gi
          requests:
            cpu: 100m
            memory: 32Mi
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:           # 해당 오브젝트의 상세한 상태 
  availableReplicas: 2
  conditions:
  - lastTransitionTime: "2023-01-12T13:11:58Z"
    lastUpdateTime: "2023-01-12T13:12:00Z"
    message: ReplicaSet "lines-admin-nextjs-deployment-5f85b84f87" has successfully
      progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  - lastTransitionTime: "2023-02-24T14:56:08Z"
    lastUpdateTime: "2023-02-24T14:56:08Z"
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  observedGeneration: 3
  readyReplicas: 2
  replicas: 2
  updatedReplicas: 2
```

### Pod를 정의하는 간단한 YAML 작성하기

```yaml
apiVersion: v1                # 디스크립터는 쿠버네티스 API 버전 v1을 준수함. 
kind: Pod                     # 오브젝트 종류가 파드임 
metadata:
  name: lines-admin-nextjs    # 파드 이름 
  labels:
    name: lines-admin-nextjs
spec:
  containers:                 # 컨테이너를 만드는 컨테이너 이미지 
  - name: lines-admin-nextjs  # 컨테이너 이름 
    image: gcr.io/google-containers/busybox
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
    ports:
      - containerPort: 8080  # 어플리케이션 수신 포트 
``` 

### 컨테이너 포트 지정

포트 정의 안에서 포트를 지정해둔 것은 단지 정보에 불과하다. 이를 생략해도 다른 클라이언트에서 포트를 통해서 파드에 연결할 수 있는 여부에 영향을 미치지 않는다.  
컨테이너가 0.0.0.0 주소에 열어 둔 포트를 통해 접속을 허용할 경우 파드 스펙에 포트를 명시적으로 나열하지 않아도 다른 파드에서 항상 해당 파트에 접속할 수 있다.
그러나 포트를 명시적으로 정의한다면, 클러스터를 사용하는 모든 사람이 파드에 노출한 포트를 빠르게 볼 수 있다.

- kubectl explain을 이용해 사용 가능한 API 오브젝트 필드 찾기

```shell
kubectl explain {오브젝트명}
```

- kubectl logs를 통해서 파드 로그 가져오기

```shell
kubectl logs lines-admin-nextjs
```

```shell
$ k logs lines-admin-nextjs-deployment-5f85b84f87-mxz7b
```

> 컨테이너 로그는 하루 단위로, 로그 파일이 10MB 크기에 도달할 때마다 순환된다. kubectl logs 명령은 마지막으로 순환된 로그 항목만 보여준다.

- 컨테이너 이름을 지정해 다중 컨테이너 파드에서 로그 가져오기

```shell
kubectl logs lines-admin-nextjs -c lines-cluster
``` 

### 파드에 요청 보내기 ( feat. Port Forward)

```shell
$ k port-forward service/lines-admin-nextjs-service 50101:3000
```

- 다양한 port-forward 방식
    - [Port Forward 를 통한 Application Cluster 에 접근하는 방법](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster)

## 레이블을 이용한 파드 구성

기본적으로 MSA 환경에서는 다양한 도메인의 서비스가 기동되어 사용되기 때문에, 각 Workloads 별로의 파드 수 증가는 파드를 분류할 필요가 생기게 된다.  
예를 들어 마이크로서비스 아키텍처의 경우 배포된 마이크로 서비스의 수는 매우 쉽게 20개를 초과한다. 이러한 구성요소는 복제돼 여러 버전 혹은 릴리즈가 동시에  
실행된다. 이로 인해 시스템에 수백 개 파드가 생길 수 있다. 파드를 정리하는 메커니즘이 없다면 너무 크고 이해하기 어려운 난장판이 되기 싶다.

### 레이블 소개

레이블은 파드와 모든 다른 쿠버네티스 리소스를 조직화 할 수 있는 단순하면서 강력한 쿠버네티스 기능이다. 레이블은 리소스에 첨부하는 키-값 쌍으로, 이 쌍은
레이블 셀렉터를 사용해 리소스를 선택할 때 활용된다.

- label 확인하기

```shell
$ k get po --show-labels
NAME                                             READY   STATUS    RESTARTS   AGE   LABELS
lines-admin-nextjs-deployment-5f85b84f87-mxz7b   1/1     Running   0          36h   app=lines-admin-nextjs,pod-template-hash=5f85b84f87
lines-admin-nextjs-deployment-5f85b84f87-nzb99   1/1     Running   0          36h   app=lines-admin-nextjs,pod-template-hash=5f85b84f87
```

- labels 이 적용되어 있는 Pod 생성하기

```yaml
apiVersion: v1 
kind: Pod 
metadata: 
  name: lines-sample 
  labels:
    creation_method: manual 
    env: prod
spec: 
  containers: 
    - image: luksa/luksa 
      name: kubia 
      ports: 
      - containerPort: 8080 
        protocol: TCP 
```

```shell
$  k create -f 007_kuberntes_in_action/p132_pod_yaml/lines_sample.yaml
```

```shell
$ k get po -L creation_method,env 
NAME                                             READY   STATUS    RESTARTS   AGE   CREATION_METHOD   ENV
lines-admin-nextjs-deployment-5f85b84f87-mxz7b   1/1     Running   0          36h                     
lines-admin-nextjs-deployment-5f85b84f87-nzb99   1/1     Running   0          36h                     
lines-sample                                     1/1     Running   0          9s    manual 
```

## 레이블 셀렉터를 이용한 파드 부분 집합 나열

- 특정한 키를 포함하거나 포함하지 않는 레이블
- 특정한 키와 값을 가진 레이블
- 특정한 키를 갖고 있지만, 다른 값을 가진 레이블

```shell
$ k get po -l creation_method=manual
NAME           READY   STATUS    RESTARTS   AGE
lines-sample   1/1     Running   0          18s
```

```shell
$ k get po -l env --show-label
NAME           READY   STATUS    RESTARTS   AGE   LABELS
lines-sample   1/1     Running   0          45s   creation_method=manual,env=prod
```

```shell
$  k get po -l '!env'    
NAME                                             READY   STATUS    RESTARTS   AGE
lines-admin-nextjs-deployment-5f85b84f87-mxz7b   1/1     Running   0          37h
lines-admin-nextjs-deployment-5f85b84f87-nzb99   1/1     Running   0          37h
```

> bash 쉘이 인식되는 기준으로 !env가 동작하지 않도록 느낌표를 처리하지 않도록 한다.

셀렉터는 쉼표는 구분된 여러 기준을 포함하는 것도 가능하다. 셀렉터를 통해 선택하기 위해서는 리소스가 모든 기준을 만족해야 한다.  
예를 들어 제품 카탈로크 마이크로 서비스의 베타 릴리스인 파드를 선택하기 위해서는 app=pc,rel=beta 셀렉터를 사용한다.
