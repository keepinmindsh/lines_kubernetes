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

### 컨테이너 포트 지정 ( ~ p124 )

 