# Section 1

## 쿠버네티스 어원 

'쿠버네티스'는 조종사, 조타수(선박의 핸들을 잡고 있는 사람)를 뜻하는 그리스어이다. 

## 쿠버네티스와 같은 시스템이 필요한 이유 

기존의 모놀리틱한 서비스 구조에서는 수직 확장이 수평확장보다 쉬웠다. 하지만 서비스 간의 도메인 복잡도가 증가하고, 사용자의 사용량이 BtoC로 확장될 경우 
수직확장 만으로는 서비스의 부하를 감당하는데는 상당한 부담이 발생할 수 있다. 모놀리틱한 서비스를 마이크로 서비스나 모듈형으로 분리하는 것은 자원의 효율적인 사용, 
서비스의 확장 등을 위해서 필수적인 요소가 되었다. 여기에서 마이크로 서비스로 개별 도메인단위로 분리된 서비스를 어떻게 하면 동일 환경에서 빠르게 확장하고 
테스트하고, 개발할 수 있을 것인가에 대한 고민들로부터 나온 것이 Container 라는 기반 환경이다. 컨테이너에 해당 서비스가 동작할 수 있는 환경을 구성하여 
쉽게 개발, 테스트, 배포가 가능하졌지만, 수십개의 컨테이너를 엔지니어가 일일이 관리 운영하기란 쉽지 않은 문제였다. 
 서비스 확장, 네트워크 구성, 로깅, 서비스 간의 영향도 관리 등을 효율적으로 하기 위해서 Service Orchestrator로써 역할을 수행할 수 있는 쿠버네티스는 
복잡한 도메인으로 구성된 마이크로 서비스와 클라우드 환경에서 필수적인 구성요소로 부각되었다. 

- 마이크로서비스 확장 
- 마이크로서비스 배포 
- 환경 요구 사항의 다양성 
- 애플리케이션에 일관된 환경 제공
- 지속적인 배포로 전환 

## 컨테이너의 이해 

### Kubernetes 의 이해

- 개발자가 애플리케이션 핵심 기능에 집중 할 수 있도록 지원
- 운영 팀이 효과적으로 리소스를 활용할 수 있도록 지원

#### 클러스터의 개념 이해하기

각 노드는 도커, Kubelet, kube-proxy 를 실행한다. Kubectl 클라이언트 명령어는 마스터 노드에서 실행 중인
쿠버네티스 API 서버로 REST 요청을 보내 클러스터와 상호작용 한다.

#### 마스터 노드 ( 컨트롤 플레인 )

전체 쿠버네티스 시스템을 제어하고 관리하는 쿠버네티스 컨트롤 플레인을 실행한다.

##### 컨트롤 플레인

클러스터를 제어하고 작동 시킨다. 하나의 마스터 노드에서 실행하거나 여러 노드로 분할되고 복제돼 고가용성을 보장할 수 있는 여러 구성 요소로 구성된다.

- 쿠버 네티스 API 서버는 사용자, 컨트롤 플레인 구성 요소와 통신한다.
- 스케줄러는 애플리케이션의 배포를 담당한다.
- 컨트롤러 매니저는 구성 요소 복제본, 워커 노드 추적, 노드 장애 처리 등과 같은 클러스터 단의 기능을 수행한다.
- Etcd는 클러스터 구성을 지속적으로 저장하는 신뢰할 수 있는 분산 데이터 저장소다.

#### 워커 노드

실제 배포되는 애플리케이션을 실행한다.

- 컨테이너를 실행하는 도커, rkt 또는 다른 컨테이너 런타임
- API 서버와 통신하고 노드의 컨테이너를 관리하는 Kubelet
- 애플리케이션 구성 요소 간에 네트워크 트래픽을 로드 밸런싱 하는 쿠버네티스 서비스 프록시

#### 쿠버네티스에서 애플리케이션 실행 

쿠버네티스에서 애플리케이션을 실행하려면 먼저 애플리케이션을 하나 이상의 컨테이너 이미지로 패키징하고 해당 이미지를 이미지 레지스트리로 푸시한 다음 
쿠버네티스 API 서버에 애플리케이션 디스크립션을 게시해야한다. 

- 디스크립션으로 컨테이너를 실행하는 방법 이해

API 서버가 애플리케이션 디스크립션을 처리할 때 스케쥴러는 각 컨테이너에 필요한 리소스를 계산하고 해당 시점에 각 노드에 할당되지 않는 리소스를 기반으로 사용 
가능한 워크 노트에 지정된 컨테이너를 할당한다. 그런 다음 해당 노드의 Kubelet은 컨테이너 런타임에 필요한 컨테이너 이미지를 가져와 컨테이너를 실행하도록 지시한다. 

- 실행된 컨테이너 유지

애플리케이션이 실행되면 쿠버네티스는 애플리케이션의 배포 상태가 사용자가 제공한 디스크립션과 일치하는지 지속적으로 확인한다. 예를 들어 항상 다섯 개의 웹 서버 인스턴스 
를 실행하도록 지정하면 쿠버네티스는 항상 정확히 다섯 개의 인스턴스를 계속 실행한다. 프로세스가 중단되거나 응답이 중지될 때와 같이 인스턴스가 제대로 작동하지 않으면 
쿠버네티스가 자동으로 다시 시작한다.

- 복제본수 스케일링 
- 이동한 애플리케이션 접근하기 

### 쿠버네티스 사용의 장점

- 애플리케이션 배포의 단순화 
- 하드웨어 활용도 높이기

서버에 수동으로 애플리케이션을 실행하는 대신 쿠버네티스를 설정하고 애플리케이션을 실행함으로써, 인프라와 애플리케이션을 분리 할 수 있다. 쿠버네티스에 애플리케이션을 
실행하도록 지시하면 애플리케이션의 리소스 요구 사항

- 상태 확인과 자가치유 

쿠버네티스는 애플리케이션 구성 요소와 이 애플리케이션이 구동 중인 노드를 모니터링하다가 노드 장애 발생시 자동으로 애플리케이션을 다른 노드로 스케쥴링 한다. 
이로써 운영 팀은 애플리케이션 구성 요소를 수동으로 마이크레이션할 필요가 없어지고, 애플리케이션을 재배치하는 대신 즉시 노드 자체를 수정해 사용 가능한 하드웨어 
리소스를 풀에 반환하는데 집중할 수 있다. 

- 오토스케일링 
- 애플리케이션 개발 단순화 

애플리케이션이 개발과 프로덕션 환경이 모두 동일한 환경에서 실행된다는 사실로 돌아가보면 버그가 발견됐을 때 큰 효과가 있다. 
버그를 빨리 발견할수록 버그를 수정하는 것이 쉽고, 수정에 더 적은 작업이 필요하다는 데 동의할 것이다. 버그를 해결하는 개발자의 작업이 줄어든다. 

# Section 2 

```shell
$ docker run busybox echo "Hello world"
```

## 기본적인 dockerfile 

```dockerfile
FROM node:7 
ADD app.js /app.js 
ENTRYPOINT ["node", "app.js"]
```

### 컨테이너 이미지 생성 

```shell
$ docker build -t kubia . 
```

> 빌드 디렉터리에 불필요한 파일을 포함시키지 마라. 특히 원격 머신에 도커 데몬이 위치한 경우 이러한 파일이 빌드 프로세스의 속도 저하를 가져온다.

### 이미지 레이어 

서로 다른 이미지가 여러 개의 레이어를 공유할 수 있기 때문에 이미지의 저장과 전송에 효과적이다. 예를 들어 

- 동일한 기본이미지를 바탕으로 다수의 이미지를 생성하더라도 기본 이미지를 구성하는 모든 레이어는 단 한번만 저장될 것이다. 
- 컴퓨터에 여러 개의 레이어가 이미 저장돼 있다면 도커는 저장되지 않은 레이어만 다운로드 한다. 

각 Dockerfile이 새로운 레이어를 하나만 생성하다고 생각할 수 있지만 그렇지 않다.  
이미지를 빌드하는 동안 기본 이미지의 모든 레이어를 가져온다음, 도커는 그 위에 새로운 레이어를 생성하고 app.js 파일을 그위에 추가한다.  
그런 다음 이미지가 실행할 때 수행돼야 할 명령을 지정하는 또 하나의 레이어를 추가한다.  
이 마지막 레이어는 kubia:latest 라고 태그를 지정한다. 

# Section 3 

## 파드에 대하여 

파드 안에 있는 모든 컨테이너는 같은 노드에서 실행된다. 절대로 두 노드에 걸쳐 배포되지 않는다. 

- 컨테이너는 단일 프로세스를 실행하는 것을 목적으로 설계했다. 
  - 단일 컨테이너에서 관련 없는 다른 프로세스를 실행하는 경우 모든 프로세스를 실행하고 로그를 관리하는 것은 모두 사용자 책임이다. 일례로 개별 프로세스가 실패하는 경우 자동으로 재시작하는 메커니즘을 포함해야 한다.  

- 여러 프로세스를 단일 구조로 묶지 않기 때문에, 컨테이너를 함께 묶고 하나의 단위로 관리할 수 있는 또 다른 상위 구조가 필요하다. 

### 같은 파드에서 컨테이너간 부분 격리 

파드의 모든 컨테이너는 동일한 네트워크 네임스페이스와 UTS 네임스페이스 안에서 실행되기 때문에, 모든 컨테이너는 같은 호스트 이름과 네트워크 인터페이스를 공유한다.  
파드의 모든 컨테이ㄴ는 동일한 IPC 네임스페이스 아래에서 실행돼 IPC를 통해 서로 통신할 수 있다.  

### 컨테이너가 동일한 IP와 포트 공간을 공유하는 방법

파드 안의 컨테이너가 동일한 네트워크 네임스페이스에서 실행되기 때문에, 동일한 IP 주소와 포트 공간을 공유한다는 것이다.  
이는 동일한 파드 컨테이너에서 실행중인 프로세스가 같은 포트 번호를 사용하지 않도록 주의해야함을 의미한다. 

### 파드 간 플랫 네트워크 소개 

쿠버네티스의 모든 파드는 하나의 플랫한 공유 네트워크 주소 공간에 상주하므로 모든 파드는 다른 파드의 IP 주소를 사용해 접근하는 것이 가능하다. 
둘 사이에서는 어떠한 NAT 도 존재하지 않는다. 두 파드가 서로 네트워크 패킷을 보내면, 상대방의 실제 IP 주소를 패킷 안에 있는 출발지 IP 주소에서 찾을 수 있다. 

### 파드에서 컨테이너의 적절한 구성 

#### 다계층 애플리케이션을 여러 파드로 분할 
#### 개별확장이 가능하도록 여러 파드로 분할  
#### 파드에서 여러 컨테이너를 사용하는 경우 

- 컨테이너를 함께 실행해야 하는가, 혹슨 서로 다른 호스트에서 실행할 수 있느낙? 
- 여러 컨테이너가 모여 하나의 구성 요소를 나타내는가, 혹은 개별적인 구성요소 인가? 
- 컨테이너가 함께, 혹은 개별적으로 스케일링돼야 하는가? 

**컨테이너는 여러 프로세스를 실행하지 말아야 한다. 파드는 동일한 머신에서 실행할 필요가 없다면 여러 컨테이너를 포함하지 말아야 한다.**

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

- 컨테이너 이름을 지정해 다중 컨테이너ㅇ 파드에서 로그 가져오기 

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
예를 들어 마이크로서비스 아키텍처의 경우 배포된 마이크로 서비스의 수는 매우 쉽게 20개를 초과한다. 이러한 구성요소는 복제돼 여러 버전 혹은 릴시스가 동시에  
실행된다. 이로 인해 시스템에 수백 개 파드가 생길 수 있다. 파드를 정리하는 메커니즘이 없다면 너무 크고 잏해하기 어려운 난장판이 되기 싶다. 

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


# Section 4 

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

쿠버네티스 테임스페이스는 오브젝트 이름의 범위를 제공한다. 모든 리소스를 하나의 단일 네임 스페이스에 두는 대신에 여러 네임스페이스로 분할 할 수 있으며, 이렇게 
분리된 네임스페이스는 같은 리소스 이름을 다른 네임스페이스에 걸쳐 여러번 사용할 수 있게 해준다.  

### 필요한 부분은 

여러 네임스페이스를 사용하면 많은 구성 요소를 가진 복잡한 시스템을 좀 더 작읍 개별 그룹으로 분리할 수 있다. 또한 멀티 테넌스 환경 처럼 리소스를 분리하는데 사용된다.  
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
컨테이너의 주 프로세스에 크래스가 발생ㅇ하면 Kubelet이 컨테이너를 다시 시작한다. 만약 여러분의 애플리케이션에 버그가 있어 가끔식 크래시가 발생하는 경우 
쿠버네티스가 애플리케이션을 자동으로 다시 시작하므로, 애플리케이션에서 특별한 작업을 하지 않더라도 쿠버네티스에서 애플리케이션을 다시 시행하는 것만으로도 
자동으로 치유할 수 있는 능력이 주어진다. 

# Section 5 

## 레플리카셋 

### 레플리카셋과 리플리케이션 컨트롤러 비교 

레플리카 셋은 레플리케이션 컨트롤러와 똑같이 동작하지만 좀 더 풍부한 표현식을 사용하는 파드 셀렉터를 갖고 있다. 레플리케이션컨터롤러의 레이블 셀렉터는 특정 레이블이 있는 파드만 
매칭 시킬 수 있는 방면, 레플리카셋의 셀렉터는 특정 레이블이 없는 파드나 레이블의 값과 상관 없이 특정 레이블의 키를 갖는 파드를 매칭 시킬 수 있다.  

예를들어 차이점을 정리해보면    

- 레플리케이션 컨트롤러 
  - 하나의 레플리케이션 컨트롤러는 레이블이 env=production 인 파드와 레이블이 env=devel를 매칭 시킬수 있다.
  - 레플리케이션 컨트롤러는 레이블의 키로 파드를 매칭시킬 수 없다.
- 레플리카셋 
  - 레플리카셋은 하나의 레플리카셋으로 두 파드 세트 모두를 매칭시켜 하나의 그룹으로 취급할 수 있다.
  - 레플리카셋은 레이블의 키값으로 파드를 매칭시킬 수 있다. 
    - 예) env 키가 있는 레이블을 갖는 모든 파드를 매핑 시킬 수 있다. ( env=* )

  
```yaml
apiVersion: apps/v1beta2
kind: ReplicaSet
metadata:
  name: kubia
spec:
  replicas: 3
  selector:
    matchLabels:
      app: kubia
  template:
    metadata:
      labels:
        app: kubia
    spec:
      containers:
      - name: kubia
        image: luksa/kubia 
```

- selector ~ : 여기서는 레플리케이션컨트롤러와 유사한 간단한 matchLabels 셀렉터를 사용한다. 
- template ~ : 템플릿은 레플리케이션 컨트롤러와 동일하다. 

> API 버전 속성에 대해서    
> - API 그룹 ( apps )
> - API 버전 ( v1beta2 )
> core API 그룹에 속하는 어떤 쿠버네티스 리소스들은 apiVersion 필드를 지정할 필요가 없다는 것을 책 전체에 걸쳐서 보게 될 것이다. 
> 최신 쿠버네티스 버전에서 도입된 다른 리소스는 여러 API 그룹으로 분류된다. 

### 리플리카셋 생성, 검사 

- replicaset 조회 

```shell
$ k get rs
NAME                                       DESIRED   CURRENT   READY   AGE
lines-admin-nextjs-deployment-5f85b84f87   0         0         0       58d
lines-admin-nextjs-deployment-864f94fc8d   2         2         2       3d14h
```

- 상세 정보 확인하기 

```shell
$ k describe rs
Name:           lines-admin-nextjs-deployment-5f85b84f87
Namespace:      default
Selector:       app=lines-admin-nextjs,pod-template-hash=5f85b84f87
Labels:         app=lines-admin-nextjs
                pod-template-hash=5f85b84f87
Annotations:    deployment.kubernetes.io/desired-replicas: 2
                deployment.kubernetes.io/max-replicas: 3
                deployment.kubernetes.io/revision: 1
                lines/admin-annotation: dream come true
                lines/hello-annotation: dream come true
Controlled By:  Deployment/lines-admin-nextjs-deployment
Replicas:       0 current / 0 desired
Pods Status:    0 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=lines-admin-nextjs
           pod-template-hash=5f85b84f87
  Containers:
   lines-admin-nextjs:
    Image:      gcr.io/lines-infra/lines_admin_front:v0.1.0
    Port:       3000/TCP
    Host Port:  0/TCP
    Limits:
      cpu:     1024m
      memory:  1Gi
    Requests:
      cpu:        100m
      memory:     32Mi
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:           <none>
```

### 레플리카셋의 표현적인 레이블 셀렉터 사용하기  

```yaml
selector: 
  matchExpressions: 
    - key: app 
      operator: In 
      values: 
        - kubia 
```

- In은 레이블의 값이 지정된 값 중 하나와 일치해야 한다. 
- NotIn은 레이블의 값이 지정된 값과 일치하지 않아야 한다. 
- Exists는 파드는 지정된 키를 가진 레이블이 포함돼야한다. 이 연산자를 사용할 때는 값 필드를 지정하지 않아야 한다. 
- DoesNotExist는 파다에 지정된 키를 가진 레이블이 포함돼 있지 않아야 한다. 값 필드를 지정하지 않아야 한다.  

### 레플리카셋 정리 

```yaml
$ k delete rs kubia 
```

## 데몬셋 

### 데몬셋을 사용해 각 노드에서 정확히 한개의 파드 실행 

클러스터의 모든 노드에, 노드당 하나의 파드가 실행되길 원하는 경우가 있을 수 있다.    

예) 로그수집기, 리소스 모니터, Kube-proxy 프로세스 등등 

### 데몬셋으로 모든 노드에 파드 실행하기  

레플리카셋이 클러스터에 원하는 수의 파드 복제본이 존재하는지 확인하는 반만, 데몬셋에는 원하는 복제본 수라는 개념이 없다. 
파드 셀렉터와 일치하는 파드 하나가 각 노드에서 실행중인지 확인하는 것이 데몬셋이 수행해야하는 역할이기 때문에 복제본의 개념이 필요하지 않다. 

### 예제를 사용한 데몬셋 설명 

SSD를 갖는 모든 노드에서 실행돼애햐난 ssd-monitor 라는 데몬이 있다고 가정하자. SSD를 갖고 있다고 표시된 모든 노드에서 이 데몬을 실행하는 데몬셋을 만든다. 
클러스터 관리자가 이런 모든 노드에 disk=ssd 레이블을 추가했으므로 해당 레이블이 있는 노드만 선택하는 노드 셀렉터를 사용해 데몬셋을 작성한다.  

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ssd-monitor
spec:
  selector:
    matchLabels:
      app: ssd-monitor
  template:
    metadata:
      labels:
        app: ssd-monitor
    spec:
      nodeSelector:
        app: sdd-monitor
      containers:
        - name: main
          image: luksa/ssd-monitor
```

```shell
k create -f ~/sources/02_linesgits/lines_kubernetes/007_kuberntes_in_action/p191_deamonsets/deamon_sets_sample.yaml
```

### 필요한 레이블을 노드에 추가하기 

```shell
k get node 

k lebel node {node name} disk=ssd

k get po   
```

데몬셋을 삭제하거나 대상으로 타케팅된 레이블이 변경될 경우 daemonSet에 의해서 생성된 파드도 삭제된다. 

## 완료 가능한 단일 태스크를 수행하는 파드 실행  

### 잡 리소스 소개  

잡은 작업이 제대로 완료되는 것이 중요한 임시 작업에 유용하다. 관리되지 않는 파드에서 작업을 실행하고 완료될 때까지 기다릴 수 있지만 작업이 수행되는 동안 
노드에 장애가 발생하거나 파드가 노드에서 제거되는 경우 수동으로 다시 생성해야 한다. 완전히 잡을 완료하는데 몇 시간이 걸리는 경우 이 작업을 수행으로 수행한다는
것은 말이 안되는 일이다. 

### 잡 리소스 정의 

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
        - name: pi
          image: perl:5.34.0
          command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4

```

파드 스펙에서는 컨테이너에서는 실행 중인 프로세스가 종료될 때 쿠버네티스가 수행할 작업을 지정할 수 있다. 이 작업 파드 스펙 속성인 restartPolicy 로 수행하며 
기본 값은 Always 이다. 잡 파드는 무한정 실행하지 않으므로 기본정책을 사용할 수 있다. 따라서 restartPolicy를 OnFailure나 Never로 명시적으로 설정해야한다. 

### 파드를 실행한 잡 보기 

```shell
$ k get jobs 

$ k get po 
NAME                                             READY   STATUS              RESTARTS   AGE
lines-admin-nextjs-deployment-864f94fc8d-cf65q   1/1     Running             0          3d15h
lines-admin-nextjs-deployment-864f94fc8d-lmtgc   1/1     Running             0          3d15h
pi-gwdzd                                         0/1     ContainerCreating   0          11s

$ k logs pi-gwdzd
3.1415926535897932384626433832795028841971693993751058209749445923078164062862089986280348253421170679821480865132823066470938446095505822317253594081284811174502841027019385211055596446229489549303819644288109756659334461284756482337867831652712019091456485669234603486104543266482133936072602491412737245870066063155881748815209209628292540917153643678925903600113305305488204665213841469519415116094330572703657595919530921861173819326117931051185480744623799627495673518857527248912279381830119491298336733624406566430860213949463952247371907021798609437027705392171762931767523846748184676694051320005681271452635608277857713427577896091736371787214684409012249534301465495853710507922796892589235420199561121290219608640344181598136297747713099605187072113499999983729780499510597317328160963185950244594553469083026425223082533446850352619311881710100031378387528865875332083814206171776691473035982534904287554687311595628638823537875937519577818577805321712268066130019278766111959092164201989380952572010654858632788659361533818279682303019520353018529689957736225994138912497217752834791315155748572424541506959508295331168617278558890750983817546374649393192550604009277016711390098488240128583616035637076601047101819429555961989467678374494482553797747268471040475346462080466842590694912933136770289891521047521620569660240580381501935112533824300355876402474964732639141992726042699227967823547816360093417216412199245863150302861829745557067498385054945885869269956909272107975093029553211653449872027559602364806654991198818347977535663698074265425278625518184175746728909777727938000816470600161452491921732172147723501414419735685481613611573525521334757418494684385233239073941433345477624168625189835694855620992192221842725502542568876717904946016534668049886272327917860857843838279679766814541009538837863609506800642251252051173929848960841284886269456042419652850222106611863067442786220391949450471237137869609563643719172874677646575739624138908658326459958133904780275901
```

### 잡에서 여러 파드 실행하기 

- Non-parallel Jobs
normally, only one Pod is started, unless the Pod fails.
the Job is complete as soon as its Pod terminates successfully.
- Parallel Jobs with a fixed completion count:
specify a non-zero positive value for .spec.completions.
the Job represents the overall task, and is complete when there are .spec.completions successful Pods.
when using .spec.completionMode="Indexed", each Pod gets a different index in the range 0 to .spec.completions-1.
- Parallel Jobs with a work queue:
do not specify .spec.completions, default to .spec.parallelism.
the Pods must coordinate amongst themselves or an external service to determine what each should work on. For example, a Pod might fetch a batch of up to N items from the work queue.
each Pod is independently capable of determining whether or not all its peers are done, and thus that the entire Job is done.
when any Pod from the Job terminates with success, no new Pods are created.
once at least one Pod has terminated with success and all Pods are terminated, then the Job is completed with success.
once any Pod has exited with success, no other Pod should still be doing any work for this task or writing any output. They should all be in the process of exiting.

위의 영어 설명에 따라서 잡은 2개 이상의 파드 인스턴스를 생성해 병렬 또는 순차적으로 실행하도록 구성할 수 있다.


```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  completions: 5 
  template:
    spec:
      containers:
        - name: pi
          image: perl:5.34.0
          command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4

```

completions를 5로 설정하면 이 잡은 다섯개의 파드를 순차적으로 실행한다.


```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  completions: 5 
  parallelism: 2
  template:
    spec:
      containers:
        - name: pi
          image: perl:5.34.0
          command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
```

잡 파드는 하나씩 차례로 실행하는 대신 잡이 여러 파드를 병렬로 실행할 수도 있다. 

### 잡스케일링 

잡이 실행되는 동안 parallelism 속성을 변경할 수도 있다. 이것은 레플리카셋이나 레플리케이션 컨트롤러를 스케일링 하는 것과 유사하며, kubectl scale 명령을 
사용해 수행할 수 있다. 

```shell
k scale job multi-completion-batch-job --replicas 3 
```

### 잡 파드가 완료되는데 걸리는 시간 제한하기 

파드 스펙에 activeDeadlineSeconds 속성을 설정해 파드의 실행 시간을 제한할 수 있다. 파드가 이보다 오래 실행되면 시스템이 종료를 시도하고 
잡을 실패한 것으로 표시한다. 

### 잡을 주기적으로 또는 한 번 실행되도록 스케쥴링하기

쿠버네티스에서의 크론작업은 크론잡 리소스를 만들어 구성한다. 잡 실행을 위한 스케쥴은 잘 알려진 크론 형식으로 지정하므로, 일반적인 크론 작업에 익숙하다면 
금방 쿠버네티스의 크론 잡을 이해할 수 있을 것이다.  


```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: hello
              image: busybox:1.28
              imagePullPolicy: IfNotPresent
              command:
                - /bin/sh
                - -c
                - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

스케쥴 설정하기  
- 분
- 시 
- 일
- 월
- 요일

예시) 0,15,30,45 * * * * : 이는 매시간, 매일, 매월, 모든 요일의 0, 15, 30, 45 분에 실행됨을 의미한다. 

```shell
k create -f ~/sources/02_linesgits/lines_kubernetes/007_kuberntes_in_action/p200_cron_jobs/cron_jobs_sample.yaml 

k get cronjob

k delete cronjob.batch/hello
```

# Section 6 

## 서비스

쿠버네티스의 서비스는 동일한 서비스를 제공하는 파드 그룹에 지속적인 단일 접점을 만들려고 할 때 생성하는 리소스다. 각 서비스는 서비스가 존재하는 동안 절대 바뀌지 않는 IP 주소와 포트가 있다. 

### Service 구성

서비스는 다수 파드 앞에서 로드 밸런서 역할을 한다. 파드가 하나가 있으면 서비스는 이 파드 하나에 정적주소를 제공한다.
서비스를 지원하는 파드가 하나든지 그룹이든지에 관계없이 해당 파드가 클러스터 내에서 이동하면서 생성되고 삭제되며 IP가
변경되지만, 서비스는 항당 동일한 주소를 가진다.

```shell

$ kubectl apply -f helloworld.service.yaml

# 변경 
$ kubectl expose pod lines-cluster --type=LoadBalancer --name lines-admin-front-http

```

- 서비스 조회하기

```shell
kubectl get services
```

### yaml 디스크립터 

```yaml
apiVersion: v1
kind: Service
metadata:
  name: lines-admin-nextjs-service
spec:
  type: LoadBalancer
  selector:
    app: lines-admin-nextjs
  ports:
    - port: 3000
      targetPort: 3000
```

이 서비스는 포트 3000의 연결을 허용하고 각 연결을 app=lines-admin-nextjs 와 일치하는 파드의 포트 8080으로 라우팅한다. 

- 서비스 확인하기 

```shell
$ k get svc
kubernetes                   ClusterIP      10.124.0.1    <none>         443/TCP          100d
lines-admin-nextjs-service   LoadBalancer   10.124.3.83   34.27.67.137   3000:30765/TCP   60d
```

- 실행 중인 컨테이너에 원격으로 명령어 실행 

```shell
$ k exec kubia-7nog1 -- curl -s http://10.111.249.153 
```

> 더블 대시를 사용하는 이유  
> 명령어의 더블 대시(--)는 kubectl 명령줄 옵션의 끝을 의미한다. 더블 대시 뒤의 모든 것은 파드 내에서 실행돼야 하는 명령이다.  
> 명령 줄에 대시로 시작하는 인수가 없는 경우 더블 대시를 사용할 필요가 없다. 

#### 서비스의 세션 어피니티 구성 

특정 클라이언트의 모든 요청을 매번 같은 파드로 리디렉션하려면 서비스의 세션 어피니티 속성을 기본값 None 대신 ClientIP로 설정한다.  
서비스 프롲ㄱ시는 동일한 클라이언트 IP의 모든 요청을 동일한 파드로 전달한다. 

```yaml
apiVersion: v1 
kind: Service 
spec: 
  sessionAffinity: ClientIP 
  ...
```

#### 동일한 서비스에서 여러 개의 포트 노출 
```yaml
apiVersion: v1 
kind: Service 
metadata:
    name: kubia 
spec: 
  ports:
    - name: http 
      port: 80 
      targetPort: 8080
    - name: https
      port: 443 
      targetPort: 8443
```

#### 이름이 지정된 포트 사용 
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
  - name: kubia
    image: luksa/kubia
    resources:
      limits:
        memory: "128Mi"
        cpu: "128m"
    ports:
      - name: http 
        containerPort: 8080
      - name: https
        containerPort: 8443
```

#### 서비스에 이름이 지정된 포트 참조하기 

- 포트 80은 http라는 컨테이너 포트에 매핑된다. 
- 포트 443은 컨테이너 포트의 이름이 https인 것에 매핑된다. 

```yaml
apiVersion: v1 
kind: Service 
spec: 
  ports:
    - name: http 
      port: 80 
      targetPort: http
    - name: https
      port: 443 
      targetPort: https
```

#### 환경 변수를 통한 서비스 검색 

```yaml
$ k exec lines-admin-nextjs-deployment-864f94fc8d-j4bkj env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=lines-admin-nextjs-deployment-864f94fc8d-j4bkj
NODE_VERSION=19.3.0
YARN_VERSION=1.22.19
NODE_ENV=production
PORT=3000
KUBERNETES_SERVICE_HOST=10.124.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT_443_TCP_PROTO=tcp
LINES_ADMIN_NEXTJS_SERVICE_SERVICE_HOST=10.124.3.83
LINES_ADMIN_NEXTJS_SERVICE_PORT_3000_TCP_PROTO=tcp
KUBERNETES_PORT=tcp://10.124.0.1:443
KUBERNETES_PORT_443_TCP_PORT=443
LINES_ADMIN_NEXTJS_SERVICE_PORT_3000_TCP=tcp://10.124.3.83:3000
LINES_ADMIN_NEXTJS_SERVICE_PORT_3000_TCP_PORT=3000
LINES_ADMIN_NEXTJS_SERVICE_PORT_3000_TCP_ADDR=10.124.3.83
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP=tcp://10.124.0.1:443
KUBERNETES_PORT_443_TCP_ADDR=10.124.0.1
LINES_ADMIN_NEXTJS_SERVICE_SERVICE_PORT=3000
LINES_ADMIN_NEXTJS_SERVICE_PORT=tcp://10.124.3.83:3000
HOME=/home/bloguser
```



#### DNS를 통한 서비스 검색 

파드가 내부 DNS 서버에서 DNS 항목을 가져오고 서비스 이름을 알고 있는 클라이언트 파드는 환경 변수 대신 FQDN으로 액세스 할 수 있다. 

#### FQDN을 통한 서비스 연결

```
backend-database.default.svc.cluster.local 
```

backend-database는 서비스 이름이고 default는 서비스가 정의된 네임스페이스를 나타내며, svc.cluster.local은 모든 클러스터의 로컬 서비스 이름에 사용되면 클러스터의 도메인 접미사이다. 

> 클라이언트는 여전히 서비스의 포트 번호를 알아야 한다. 서비스가 표준 포트( 80, 5432 )를 사용하는 경우 문제가 되지 않는다.  
> 그렇지 않은 경우 크라리언트는 환경 변수에서 포트 번호를 얻을 수 있어야 한다. 

서비스에 연결하는 것은 훨씬 간단한다. 프론트엔드 파드가 데이터베이스 파드와 동일한 네임스페이스에 있을 경우 svc.cluster.local 접시마와 테임스페이스는 생략할 수 있다. 
따라서 서비스를 단순히 backend-database라고 할수 있다. 

#### 파드 컨테이너 내에서 셸 실행 

> 이 작업을 수행하려면 컨테이너 이미지에서 셸 바이너리 실행 파일이 사용 가능해야 한다. 

```shell
$ k exec -it {pod name} bash 

$ curl http://kubia.default.svc.cluster.local 

$ curl http://kubia.default 

$ curl http://kubia 

# 위처럼 요청 url에서 서비스 이름을 사용해 서비스에 엑세스 할 수 있다. 각 파드 컨테이너 내부의 DNS Resolver가 구성돼 있기 때문에 네임스페이스와 svc.cluster.local 접미사를 생략할 수 있다. 
# 컨테이너에서 /ect/resolv.conf 파일을 보면 이해할 수 있다. 

$ cat /etc/resolv.conf
```

#### 서비스에 핑을 할수 없는 이유 

```shell
$ ping kubia 
```

서비스로 curl은 동작하지만 핑은 응답이 없다. 이는 서비스의 클러스터 IP가 가장 IP이므로 서비스 포트와 결합된 경우에만 의미가 있기 때문이다. 

### 클러스터 외부에 있는 서비스 연결 

쿠버네티스 서비스 기능으로 외부 서비스를 노출하려는 경우가 있을 수 있다. 서비스가 클러스터내에 있는 파드로 연결을 전달하는 게 아니라, 외부 IP와 포트로 연결을 전달하는 것이다. 

#### 서비스 엔드포인트 

서비스는 파드이 직접 연결되지 않는다. 대신 엔드포인트 리소스가 그 사이에 있다. 

```shell
$  k describe svc lines-admin-nextjs-service
Name:                     lines-admin-nextjs-service
Namespace:                default
Labels:                   <none>
Annotations:              cloud.google.com/neg: {"ingress":true}
Selector:                 app=lines-admin-nextjs  # 서비스의 파드 셀렉터는 엔드포인트 목록을 만드는 데 사용한다. 
Type:                     LoadBalancer
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.124.3.83
IPs:                      10.124.3.83
LoadBalancer Ingress:     34.27.67.137
Port:                     <unset>  3000/TCP
TargetPort:               3000/TCP
NodePort:                 <unset>  30765/TCP
Endpoints:                10.120.0.11:3000,10.120.3.17:3000   # 이 서비스의 엔드 포인트를 나타내는 파드 IP와 포트 목록 
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

엔드포인트 리소스는 서비스로 노출되는 파드의 IP 주소와 포트 목록이다. 엔드 포인트 리소스는 다른 쿠버네티스 리소스와 유사하므로 kubectl get을 사용해 기본 정보를 표시할 수 있다. 

```shell
$ k get endpoints lines-admin-nextjs-service
NAME                         ENDPOINTS                           AGE
lines-admin-nextjs-service   10.120.0.11:3000,10.120.3.17:3000   60d
```

#### 서비스 엔드 포인트 수동 구성 

파드 셀렉터 없이 서비스를 만들변 쿠버네티스는 엔드포인트 리소스를 만들지 못한다. 파드 셀렉터가 없어, 서비스에 포함된 파드가 무엇인지 알 수 없다. 

##### 셀렉터 없이 서비스 생성 

```yaml
apiVersion: v1 
kind: Service 
metadata: 
  name: external-service # 서비스 이름은 엔드포인트 오브젝트 이름과 일치해야 한다.  
spec:   # 이 서비스에는 셀렉터가 정의되어 있지 않다. 
  ports:
    - port: 80 
```

##### 셀렉터가 없는 서비스에 관한 엔드포인트 리소스 생성 

엔드 포인트는 별도의 리소스 이며, 서비스 속성은 아니다. 셀렉터가 없는 서비스를 생성했기 때문에 엔드포인트 리소스가 자동으로 생성되지 않는다. 

```yaml
apiVersion: v1 
kind: Endpoints 
metadata: 
  name: external-service # 엔드포인트 오브젝트의 이름은 서비스 이름과 일치해야 한다. 
subsets: 
  - addresses: 
    - ip: 11.11.11.11 # 서비스가 연결을 전달할 엔드 포인트의 IP 
    - ip: 22.22.22.22
    ports:
    - port: 80 # 엔드 포인트의 대상 포트  
      
```

엔드 포인트 오브젝트는 서비스와 이름이 같아야 하고 서비스를 제공하는 대상 IP 주소와 포트 목록을 가져야 한다. 서비스와 엔드 포인트 리소스가 모두 서버에 게시되면 파드 셀렉터가 있는 
일반 서비스 처럼 서비스를 사용할 수 있다. 서비스가 만들어진 후 만들어진 컨테이너에는 서비스의 환경 변수가 포함되며 IP:포트 쌍에 대한 모든 연결은 서비스 엔드 포인트 간에 로드 밸런싱 한다. 


#### 외부 서비스를 위한 별칭 생성 

##### External 별칭 생성 

서비스를 사용하는 파드에섯 ㅣㄹ제 서비스 이름과 위치가 숨겨져 나중에 externalName 속성을 변경하거나 유형을 다시 ClusterIP로 변경하고 서비스 스펙을 만들어 
서비스 스펙을 수정하면 다중에 다른 서비스를 가리킬 수 있다. 

ExternalName 서비스는 DNS 레벨에서만 구현된다. 서비스에 관한 간단한 CNAME DNS 레코드가 생성된다. 

```yaml
apiVersion: v1 
kind: Service 
metadata: 
  name: external-service
spec:
  type: ExternalName 
  externalName: someapi.somecompany.com
  ports:  
  - port: 80
```

### 외부 클라이언트에 서비스 노출 

- 노드 포트로 서비스 유형 설정 
- 서비스 유형을 노드 포트로 유형의 확장인 로드밸런서로 설정 
- 단일 IP 주소로 여러 서비스를 노출하는 인그레스 리소스 만들기 

#### 노드 포트 서비스 

```yaml
apiVersion: v1 
kind: Service 
metadata:
  name: kubia-nodeport 
spec: 
  type: NodePort 
  ports:
    - port: 80 
      targetPort: 8080 
      nodePort: 30123 
  selector:
    app: kubi 
```

```shell
$ k get svc kubia-nodeport
```

##### 노드포트 서비스에 액세스 할 수 있도록 방화벽 규칙 변경 

```shell
gcloud copute firewall-rules create kubia-svc --allow=tcp:30123 
```

#### 로드밸런서 서비스 생성 

```yaml
apiVersion: v1 
kind: Service 
metadata: 
  name: kubia-loadbalancer 
spec:
  type: LoadBalancer 
  ports:
  - port: 80
    targetPort: 8080 
  selector:
    app: kubia
```

```shell
$ k get svc kubia-loadbalancer 
```

#### 외부 연결 특성의 이해 

```yaml
sepc:
  externalTrafficPolicy: Local
```

### 인그레스 리소스로 서비스 외부 노출  

인그레스 : 들어가거나 들어가는 행위, 들어갈 권리, 틀어갈 수단이나 장소, 진입로 

#### 인그레스가 필요한 이유 

로드밸런서 서비스는 자신의 공용 IP 주소를 가진 로드 밸런서가 필요하지만, 인그레스는 한 IP 주소로 수십 개의 서비스에 접근이 가능하도록 지원해준다. 
클라이언트가 HTTP 요청을 인그레스에 보낼 때, 요청한 호스트와 경로에 따라 요청을 전달할 서비스가 결정된다.  

인그레스는 네트워크 스택의 어플리케이션 계층에서 작동하며 서비스가 할 수 없는 쿠키 기반 세션 어피니티 등과 같은 기능을 제공할 수 있다. 

#### 인그레스 컨트롤러가 필요한 경우  

인그레스 오브젝트가 제공하는 기능을 살펴보기 전에 인그레스 리소스를 작동시키려면 클러스터에 인그레스 컨트롤러를 실행해야 한다. 쿠버네티스 환경마다 다른 컨트롤러 구현을 
사용할 수 있지만 일부는 기본 컨트롤러를 전혀 제공하지 않는다. 

##### Minikube에서 인그레스 애드온 활성화 

```shell
$ minikube addons list 

$ minikube addons enable ingress 

$ k get po --all-namespaces 
```

#### 인그레스 리소스 생성  

```yaml
apiVersion: extensions/v1beta1
kind: Ingress 
metadata: 
  name: kubia 
spec: 
  rules: 
  - host: kubia.example.com 
    http: 
    paths:
      - path: / 
        backend: 
          serviceName: kubia-nodeport 
          servicePort: 80
``` 

> 클라우드 공급자의 인그레스 컨트롤러는 인그레스가 노드포트 서비스를 가리킬 것을 요구한다. 하지만 그것이 쿠버네티스 자체의 요구사항은 아니다 

##### 인그레스로 서비스 액세스  

```shell
$ k get ingresses
```

##### 인그레스 컨트롤러가 구성된 호스트의 IP를 인그레스 엔드포인트로 지정 

IP를 알고나면 kubia.example.com을 해당 IP로 확인하도록 DNS 서버를 구성하거나, 다음 줄을 /etc/hosts에 추가할 수 있다. 

##### 인그레스로 파드 액세스 

```shell
$ curl http://kubia.example.com 
You've hit kubia-ke823
```

##### 인그레스 동작 방식 

- 클라인트가 kubia.example.com 을 찾는다. 
- 클라이언트는 헤더 Host:kubia.example.com 과 함께 HTTP GET 요청을 보낸다. 
- 컨트롤러가 파드에 요청을 보낸다. 

> 인그레스 컨트롤러는 요청을 서비스로 전달하지 않는다. 파드를 선택하는 데만 사용한다. 모두는 아니지만 대부분의 컨트롤러는 이와 같이 동작한다.  

#### 하나의 인그레스로 여러 서비스 노출 

인그레스 스펙을 보면 규칙과 경로가 모두 배열이므로 여러 항목을 가질 수 있다.   

##### 동일한 호스트의 다른 경로로 여러 서비스 매핑  

- kubia.example.com/kubia 으로의 요청은 kubia 서비스로 라우팅된다.

```yaml
... 
  - host: kubia.example.com 
    http: 
      paths: 
      - path: /kubia 
        backend: 
          serviceName: kubia 
          servicePort: 80 
      - path: /bar 
        backend: 
          serviceName: bar 
          servicePort: 80 
```

위의 경우 요청은 URL의 경로에 따라 두 개의 다른 서비스로 전송된다. 따라서 클라이언트는 단일 IP 로 두 개의 서비스에 도달할 수 있다. 

##### 서로 다른 호스트로 서로 다른 서비스에 매핑하기 

```yaml
spec:
  rules:
  - host: foo.example.com 
    http: 
      paths:
      - path: /
        backend:
          serviceName: foo 
          servicePort: 80 
  - host: bar.example.com 
    http: 
      paths:
      - path: / 
        backend: 
          serviceName: bar 
          servicePort: 80
```

#### TLS 트래핑을 처리하도록 인그레스 구성 

##### 인그레스를 위한 TLS 인증서 생성  

클라이언트가 인그레스 컨트롤러에 대한 TLS 연결을 하면 컨트롤러는 TLS 연결을 종료한다. 클라이언트와 컨트롤러 간의 통신은 암호화 되지만 컨트롤러와 벡엔드 파드 간의 
통신은 암호화 되지 않는다. 파드에서 실행 중인 애플리케이션은 TLS를 지원할 필요가 없다. 예를 들어 파드가 웹 서버를 실행할 경우 HTTP 트래픽만 허용하고 인그레스 컨트롤러가 TLS와 
관련된 모든 것을 처리하도록 할 수 있다. 컨트롤러가 그렇게 하려면 인증서와 개인 키를 인그레스에 첨부해야 한다. 

```shell
$ openssl genrsa -out tls.key 2048 
$ openssl req -new -x509 -key tls.key -out tls.cert -day 360 -subj 

# 만들어진 파일로 시크릿을 만든다. 
$ kubectl create secret tls tls-secret --cert=tls.cert --key=tls.key 
```

> CertificateSigningRequest 리소스로 인증서 서명
> 인증서를 직업 서명하는 대신 CSR 리소스를 만들어 인증서에 서명할 수 있다. 사용자 또는 해당 애플리케이션이 인반 인증서 요청을 생성할 수 있고 CSR에 넣으면
> 그 다음 운영자나 자동화된 프로세스가 다음과 같이 요청을 승인할 수 있다.
> kubectl certificate approve <name of the CSR>

```yaml
  apiVersion: extension/v1beta1
kind: Ingress

metadata:
  name: kubia
spec:
  tls:
  - hosts: 
  - kubia.example.com 
  secretName: tls-secret 
  rules:
  - host: kubia.exampele.com 
  http:
  paths:
  - path: / 
  backend:
  serviceName: kubia-nodeport 
  servicePort: 80  
```

- 전체 TLS 구성이 이 속성 아래에 있다. 
- kubia.example.com 호스트 이름의 TLS 연결이 허용된다. 
- 개인키와 인증서는 이전에 작성한 tls-secret을 참조한다. 

```shell
$ curl -k -v https://kubia.example.com/kubia 
# 명령어의 출력에는 애플리케이션의 응답과 인그레스에 구성한 서버 인증서가 표시된다. 
```

### 파드가 연결을 수락할 준비가 됐을 때 신호 보내기  

#### Readiness Prove  

레디니스 프로브는 주기적으로 호출되며 특정 파드가 클라이언트 요청을 수신할 수 있는지 결정한다. 컨테이너의 레디니스 프로브가 성공을 반환하면 컨테이너가 요청을 수락할 
준비가 됐다는 신호다. 준비가 됐다라는 표시는 분명히 각 컨테이너 마다 다르다. 쿠버네티스는 컨테이너에서 실행되는 애플리케이션이 간단한 GET / 요청에 응답하는지 또는 
특정 URL 경로를 호출 할 수 있는지 확인하거나 필요에 따라 애플리케이션이 준비됐는지 확인하기 위해 전체적인 항목을 검사하기도 한다.  

##### Types 

- 프로세스를 실행하는 Exec 프로브는 컨테이너의 상태를 프로세스의 종료 상태 코드로 결정한다. 
- HTTP GET 프로브는 HTTP GET 요청을 컨테이너로 보내고 응답의 HTTP 상태 코드를 보고 컨테이너가 준비됐는지 여부를 결정한다. 
- TCP 소켓 프로브는 컨테이너의 지정된 포트로 TCP 연결을 연다. 소켓이 연결되면 컨테이너가 준비된 것으로 간주한다.  

라이브니스 프로브와 달리 컨테이너가 준비 상태 점검에 실패하더라도 컨테이너가 종료되거나 다시 시작되지 않는다. 이는 라이브니스 프로브와 레디니스 프로브 사이의 중요한 차이다. 
라이브 프로브는 상태가 좋지 않은 컨테이너를 제거하고 새롭고 건강한 컨테이너로 교체해 파드의 상태를 정상으로 유지하는 반면, 레디니스 프로브는 요청을 처리할 준비가 된 파드의 컨테이너만 요청을 수진하도록 한다. 
이건은 컨테이너를 시작할 때 주로 필요하지만 컨테이너가 작동한 후에도 유용하다. 

###### 레디니스 프로브가 중요한 이유 

파드 그룹이 다른 파드에서 제공하는 서비스에 의존한다고 가정해보자. 프론트엔드 파드 중 하나에 연결 문제가 발생해 더 이상 데이터베이스에 연결할 수 없는 경우, 
해당 시점에 파드가 해당 요청을 처리할 준비가 되지 않았다는 신호를 레디니스 프로브가 쿠버네티스에게 알리는 것이 현명할 수 있다.  
다른 파드 인스턴스에 동일한 유형의 연결 문제가 발생하지 않는다면 정상적으로 요청을 처리할 수 있다. 레디니스를 사용하면 클라이언트가 정상 상태인 파드하고만 동신하고 시스템에 문제가 있다는 것을 절대 알아차리지 못한다. 

#### 파드에 레디니스 프로브 추가  

##### 파드 템플릿에 레디니스 프로브 추가 

```shell
# 기존 레플리케이션 컨트롤러의 파드 템플릿의 프로브를 추가한다. 
$ kubectl edit rc kubia 
```

```yaml
apiVersion: v1 
kind: ReplicationController 
```

실제 샘플 

> [Readiness & Liveness Prove](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: goproxy
  labels:
    app: goproxy
spec:
  containers:
    - name: goproxy
      image: registry.k8s.io/goproxy:0.1
      ports:
        - containerPort: 8080
      readinessProbe:
        tcpSocket:
          port: 8080
        initialDelaySeconds: 5
        periodSeconds: 10
      livenessProbe:
        tcpSocket:
          port: 8080
        initialDelaySeconds: 15
        periodSeconds: 20
```

```shell
$ k get po 햐
# k exec 명령어로 파드의 컨테이너 내에 touch 명령어가 실행되게 되면 해당 시점에 Pod가 준비상태로 표시된다. 
$ k exec kubia-2r1qp -- touch /var/ready 

$ k get po kubia-2r1qb 
```

#### 실제 환경에서 레디니스 프로브가 수행해야 하는 기능 

> 서비스에서 파드를 수동으로 추가하거나 제거하려면 파드와 서비스의 레이블 셀렉터에 enabled=true 레이블을 추가한다. 
> 서비스에서 파드를 제거하려면 레이블을 제거하라.  

##### 레디니스 프로브를 항상 정의 하라 

파드에 레디니스 프로브를 추가하지 않으면 파드가 시작하는 즉시 서비스 엔드 포인트가 된다. 
애플리케이션 수신 연결을 시작하는데 너무 오래 걸리는 경우 클라이언트의 서비스 요청은 여전히 시작단계로 수신연결은 수락할 준비가 되지 않은 상태에서 
파드로 전달된다. 따라서 클라이언트는 Connnection Refuxed 유형의 에러를 보게 된다. 

> 기본 URL에 HTTP 요청을 보내더라도 항상 레디니스 프로브를 정의해야 한다. 

##### 레디니스 프로브에 파드의 종료 코드를 포함하지 마라. 

쿠버네티스는 파드를 삭제하자마자 모든 서비스에서 파드를 제거하기 때문에 파드 종료 코드를 포함해서는 안된다.!  

### 헤드리스 서비스로 개별 파드 찾기  

#### 헤드리스 서비스 생성 

서비스 스펙의 ClusterIP 필드를 None으로 설정하면 쿠버네티스는 클라이언트가 서비스의 파드에 연결할 수 있는 클러스터 IP를 할당하지 않기 때문에 서비스가 헤드리스 상태가 된다. 

```yaml
apiVersion: v1 
kind: Service 
metadata: 
  name: kubia-headless 
spec:
  clusterIp: None 
  ports: 
  - port: 80 
    targetPort: 8080 
  selector: 
    app: kubia 
```

#### DNS로 파드 찾기 

##### YAML 매니페스트를 쓰지 않고 파드 실행 

```shell
$ k run dnsutils --image=tutum/dnsutils --generator=run-pod/v1 --command --sleep infinity
```

##### 헤드리스 서비스를 위해 반한된 DNS A 레코드 

```shell
$ k exec dnsutils nslookup kubia-headless 
```

```shell
$ k exec dnsutils nslookup kubia 
```

#### 모든 파드 검색 - 준비 되지 않은 파드도 포함 

```yaml
kind: Service 
metadata:
  annotations:
    service.alpha.kubernetes.io/tolerate-unready-endpoints: "true"
```

### 서비스 문제 해결 

- 먼저 외부가 아닌 클러스터 내에서 서비스의 클러스터 IP에 연결되는지 확인한다. 
- 서비스에 액세스할 수 있는지 확인하려고 서비스 IP로 핑을 할 필요 없다. 
- 레디니스 프로브를 정으했다면 정의했다면 성공했즞니 확인하라. 그렇지 않으면 파드는 서비스에 포함되지 않는다. 
- 파드가 서비스의 일부인지 확인하려면 k get endpoints 를 사용해 해당 엔드 포인트 오브젝트를 확인한다. 
- FQDN이나 그 일부  서비스에 액세스 하려고 하는데 동작하지 않는 경우, FQDN 대신 클러스터 IP를 사용해 액세스 할 수 있는지 확인한다. 
- 대상 포트가 아닌 서비스로 노출된 포트에 연결하고 있는지 확인한다. 
- 파드 IP에 직접 연결해 파드가 올바른 포트에 연결돼 있는지 확인한다. 
- 파드 IP로 애플리케이션에 액세스 할 수 없는 경우 애플리케이션이 로컬 호스트에만 바인딩하고 있는지 확인한다.  

# Section 7 

## Volume

쿠버네티스 볼륨은 파드의 구성 요소로 컨테이너와 동일하게 파드 스펙에서 정의된다. 볼륨은 독립적인 쿠버네티스 오브젝트가 아니므로 자체적으로 생성, 삭제될 수 앖다. 
볼륨은 파드의 모든 컨테이너에서 사용 가능하지만 접근하려면 컨테이너에서 각각 마운트돼야 한다. 각 컨테이너에서 파일시스템의 어느 경로에나 불륨을 마운트할 수 있다. 


