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