# Kubernetes - Hello World 

- 기동할 App.js 만들기 
```javascript
consthttp = require('http')
constos = require('os')

console.log("Kubia server starting...")

consthandler =function(request, response) {
    console.log("Received request from " + request.connection.remoteAddress);
    response.writeHead(200)
    response.end("You've hit " + os.hostname() + "\n")
};

constwww = http.createServer(handler);
www.listen(8080)
```

- Dockerfile 만들기 
```dockerfile
FROM node:7
ADD app.js/app.js

ENTRYPOINT["node", "app.js"]
```

- Docker Image 빌드 하기 
```shell
docker build . -t lines_appjs -f ~/sources/02_linesgits/lines_kubernetes/008_dockerfiles/213_dockerfilesamples/Dockerfile
```

- Docker Container 기동하기 
```shell
docker run --name lines-appjs-container -p 8080:8080 -d lines_appjs
```

- 컨테이너 내부 정보 세세히 살피기
```shell
 docker inspect lines-appjs-container
```

# Kubernetes의 이해 

- 개발자가 애플리케이션 핵심 기능에 집중 할 수 있도록 지원 
- 운영 팀이 효과적으로 리소스를 활용할 수 있도록 지원

## 마스터 노드 ( 컨트롤 플레인 ) 

전체 쿠버네티스 시스템을 제어하고 관리하는 쿠버네티스 컨트롤 플레인을 실행한다.

### 컨트롤 플레인 

클러스터를 제어하고 작동 시킨다. 하나의 마스터 노드에서 실행하거나 여러 노드로 분할되고 복제돼 고가용성을 보장할 수 있는 여러 구성 요소로 구성된다. 

- 쿠버 네티스 API 서버는 사용자, 컨트롤 플레인 구성 요소와 통신한다. 
- 스케줄러는 애플리케이션의 배포를 담당한다. 
- 컨트롤러 매니저는 구성 요소 복제본, 워커 노드 추적, 노드 장애 처리 등과 같은 클러스터 단의 기능을 수행한다. 
- Etcd는 클러스터 구성을 지속적으로 저장하는 신뢰할 수 있는 분산 데이터 저장소다. 

## 워커 노드 

실제 배포되는 애플리케이션을 실행한다. 

- 컨테이너를 실행하는 도커, rkt 또는 다른 컨테이너 런타임 
- API 서버와 통신하고 노드의 컨테이너를 관리하는 Kubelet 
- 애플리케이션 구성 요소 간에 네트워크 트래픽을 로드 밸런싱 하는 쿠버네티스 서비스 프록시 

# Pod

## Lifecycle

### Pod Lifecycle 

Pod 의 기본적인 라이프사이클은 Pending Phase 에서 시작해서, 만약 주요한 컨테이너의 최소 한 개의 상태가 OK로 시작되면  Running Phase 로 변경된다.  
그런 다음 파드의 컨테이너가 실패로 종료되었는지 여부에 따라 Succeeded 또는 Failed 로 변경된다. 

- **Pending** 
  - 파드가 쿠버네티스 클러스터에서 승인되었지만, 하나 이상의 컨테이너가 설정되지 않았고 실행할 준비가 되지 않았다. 여기에는 파드가 스케줄되기 이전까지의 시간 뿐만 아니라 네트워크를 통한 컨테이너 이미지 다운로드 시간도 포함된다.
    - ReadinessProbe
    - Policy
- **Running**
  - 파드가 노드에 바인딩되었고, 모든 컨테이너가 생성되었다. 적어도 하나의 컨테이너가 아직 실행 중이거나, 시작 또는 재시작 중에 있다.
    - LivenessProbe 
    - Qos 
- **Succeeded**
  - 파드에 있는 모든 컨테이너들이 성공적으로 종료되었고, 재시작되지 않을 것이다.
    - Policy
- **Failed**
  - 파드에 있는 모든 컨테이너가 종료되었고, 적어도 하나 이상의 컨테이너가 실패로 종료되었다. 즉, 해당 컨테이너는 non-zero 상태로 빠져나왔거나(exited) 시스템에 의해서 종료(terminated)되었다.
    - Policy
- **Unknown**
  - 어떤 이유에 의해서 파드의 상태를 얻을 수 없다. 이 단계는 일반적으로 파드가 실행되어야 하는 노드와의 통신 오류로 인해 발생한다.

### Container Status 

- **Waiting** 
  - 만약 컨테이너가 Running 또는 Terminated 상태가 아니면, Waiting 상태이다. Waiting 상태의 컨테이너는 시작을 완료하는 데 필요한 작업(예를 들어, 컨테이너 이미지 레지스트리에서 컨테이너 이미지 가져오거나, 시크릿(Secret) 데이터를 적용하는 작업)을 계속 실행하고 있는 중이다. kubectl 을 사용하여 컨테이너가 Waiting 인 파드를 쿼리하면, 컨테이너가 해당 상태에 있는 이유를 요약하는 Reason 필드도 표시된다.

- **Running**
  - Running 상태는 컨테이너가 문제없이 실행되고 있음을 나타낸다. postStart 훅이 구성되어 있었다면, 이미 실행되고 완료되었다. kubectl 을 사용하여 컨테이너가 Running 인 파드를 쿼리하면, 컨테이너가 Running 상태에 진입한 시기에 대한 정보도 볼 수 있다.

- **Terminated**
  - Terminated 상태의 컨테이너는 실행을 시작한 다음 완료될 때까지 실행되었거나 어떤 이유로 실패했다. kubectl 을 사용하여 컨테이너가 Terminated 인 파드를 쿼리하면, 이유와 종료 코드 그리고 해당 컨테이너의 실행 기간에 대한 시작과 종료 시간이 표시된다.

### ContainerRestartPolicy 

restartPolicy 는 파드의 모든 컨테이너에 적용된다.   
restartPolicy 는 동일한 노드에서 kubelet에 의한 컨테이너 재시작만을 의미한다.   
파드의 컨테이너가 종료된 후, kubelet은 5분으로 제한되는 지수 백오프 지연(10초, 20초, 40초, …)으로 컨테이너를 재시작한다.    
컨테이너가 10분 동안 아무런 문제없이 실행되면, kubelet은 해당 컨테이너의 재시작 백오프 타이머를 재설정한다.

- **restartPolicy** 
  - Always, OnFailure, Never
  - 기본 값 : Always

### Pod Condition 

- **PodScheduled**
  - 파드가 노드에 스케줄되었다.
- **ContainersReady**
  - 파드의 모든 컨테이너가 준비되었다.
- **Initialized**
  - 모든 초기화 컨테이너가 성공적으로 완료(completed)되었다.
- **Ready**
  - 파드는 요청을 처리할 수 있으며 일치하는 모든 서비스의 로드 밸런싱 풀에 추가되어야 한다.

### References 

```yaml
status:
  phase: Pending  # Pending, Running, Succeeded, Failed, UnKnown 
  conditions: # Initialized , ContainerReady, PodScheduled, Ready 
  - type: Initialized
    status: 'True'
    lastProbeTime: null 
    lastTransitionTime: '2019-09-26T22:07:56Z'
  - type: PodScheduled 
    status: 'True'
    lastProbeTime: null 
    lastTransitionTime: '2019-09-26T22:07:56Z'
  - type: ContainersReady
    status: 'False'
    lastProbeTime: null
    lastTransitionTime: '2019-09-26T22:08:11Z'
    reason: ContainersNotReady # ContainerNotReady, PodCompleted 
  - type: Ready
    status: 'False'
    lastProbeTime: null
    lastTransitionTime: '2019-09-26T22:08:11Z'
    reason: ContainersNotReady
  containerStatuses:
  - name: container 
    state: # Waiting, Running, Terminated  
      waiting: 
        reason: ContainerCreating  # ContainerCreating, CrashLoopBackOff, Error, Completed 
    lastState: {}
    ready: false
    restartCount: 0 
    image: tmkube/init
    imageID: ''
    started: false
```

- Pod 의 최초상태는 Pending 
  - Container 가 기동되기 전에 설정되어야할 항목이 있음 
  - Pending 상태에서 Failed 로 상태가 변경될 수 있음 
  - Pending 상태에서 Unknown 으로 상태가 변경될 수 있음

```yaml
apiVersion: v1 
kind: Pod 
metadata: 
  name: myapp-pod 
  labels: 
    app: myapp
spec: 
  containers: 
  - name: myapp-container
    image: busybox:1.28 
    command: [ 'sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers: # 본 Container 보다 먼저 실행되었다고 판단하는 것 
  - name: init-myservice 
    image: busybox:1.28 
    command: [ 'sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb 
    image: busybox:1.28
    command: [ 'sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```

- Pod 의 그 다음 단계는 Running
  - Running 단게에서는 Unknown으로 변경될 수 있음
  - Unkown 상태는 Failed 상태로 변경이 가능함.

- Pod 의 마지막 상태는 Failed 또는 Succeeded 

- 마지막으로 Pod의 상태는 무조건 
  - ContainerReady : False 
  - Ready : False

### Pod ( ReadinessProbe, LivenessProbe )

기본적으로 Pod를 종료할 때 별다른 설정 없이 종료하게 되면, 종료되기 전까지의 시점에 일부 요청건에 대해서 장애가 발생할 수 있는데,   
그 이유는 기존의 Pod가 종료되기전, 신규 Pod 가 구성된 시점에는 아직 정상적동하지 않는 Pod일 수 있기 때문에 (내부의 소스 기동 시간 등등 ),  
50:50의 확률로 서비스가 정상적으로 종료가 안된 Pod로 요청이 접수 될 수 있는데, 이건 신규 Pod가 기동되는 시점도 동일하게 발생할 수 있는 문제이다. 

- ReadinessProbe 
  - App 구동 순간에 트래픽 실패를 없앰 

- LivenessProbe 
  - App 장애시 지속적인 트래픽 실패를 없앰 

![Deployments](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/ProbesSettings01.png)

![Deployments](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/ProbesSettings02.png)

## QoS classes 

설정에 따라 파드에 필요한 성능을 관리하는 방식을 제공함.  
예) 3개의 파드 중 1개의 파드에 성능이 더 필요할 경우 3개중 하나를 죽이고 파드 하나로 성능을 올려주는 역할을 설정할 수 있음.

![Deployments](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/qos_class.png)

```yaml
kind: Prod 
spec:
  containers:
    - resources:
        requests:
          memory: 1G
          cpu: 2
        limits:
          memory: 2G
          cpu: 4
```

- Guaranteed 
- Burstable 
  - OOM Score 
- BestEffort 

## Node Scheduling 

- Node 선택 : 원하는 노드로 자원 할당이 되도록 관리할 수 있음 
  - NodeName 
  - NodeSelector 
  - NodeAffinity
    - matchExpressions 
      - required, preferred 
        - key 
        - operator : Exists, DoesNotExist, In, NotIn, Gt, Lt

- Pod 간 집중/분산 
  - Pod Affinity 
    - podAffinity 
      - matchExpressions
        - key : type 
        - operator : ~ 
        - values:[~]
    - topologyKey 
  - Anti-Affinity 
    - podAntiAffinity 

- Node에 할당 제한 
  - Toleration / Taint
    - Pod 
      - Toleration 
        - key 
        - operator : Equal, Exists  
        - value 
        - effect : NoSchedule, PreferNoSchedule, NoExecute 
  - Pod가 Toleration을 설정해야 Taint로 설정된 Node로 Pod 세팅이 가능함. 

# Object - Service 

![matchExpressions](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/service.png)


## ClusterIP 

- 클러스터내 접근 가능! 
  - 인가된 사용자 ( 운영자 )
    - 내부 관리자 접근 관리의 용도 
  - 내부 대쉬보드 
  - Pod의 서비스 상태 디버깅 

```yaml
apiVersion: v1
kind: Service 
metadata: 
  name: svc-1
spec: 
  selector: 
    app: pod 
  ports: 
    - port: 9000
      target: 8080
type: ClusterIP
```

## NodePort 

- 모든 Node 에 Port 할당 
  - 내부 포트 서비스를 만들어서 서로 접근이 가능하게 할 수 있음 
- 내가 다수 노드가 있는 서비스에 대해서 1번 노드로만 전달하더라도 연결되어 있는 각 파드로 자동으로 분산되어 호출됨. 
  - 하지만 만약 내가 원하는 IP에 대해서만 설정하고 싶은 경우 
    - externalTrafficPolicy: Local 
- 내부망 연결용 
  - 데모나 임시 연결용 

```yaml
apiVersion: v1
kind: Service 
metadata: 
  name: svc-2 
spec: 
  selector: 
    app: pod 
  ports:
    - port:9000 
      tartPort: 8080 
      nodePort: 30000 #30000 ~ 32767 
type: NodePort 
```

## LoadBalancer

- 외부 접속을 할 경우 주로 많이 사용함. 
- Load Balancer는 외부 IP를 연동하기 위한 설정이 필요 
  - Plugin - GCP, AWS, Azure, OpenStack
- 외부 시스템 노출 용 

```yaml
apiVersion: v1 
kind: Service 
metadata: 
  name: svc-3 
spec: 
  selector:
    app: pod 
  ports: 
    - port: 9000 
      targetPort: 8080
type: LoadBalancer 
```

## Headless 

![matchExpressions](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/headless.png)

- FQDN 

## EndPoint

![matchExpressions](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/endpoint.png)

Selector와 Label 없이 직접 Pod에 연결하는 방식으로 만들 수 있음 

## External Name

![matchExpressions](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/externalname.png)

# Object - Volume

Kubernetes Cluster 분리해서 관리가 된다! 

- External Network
  - HostPath 
  - Local
  - On-Premise Solution 
  - NFS 
- Internal Network
  - AWS 
  - GCP 
  - Azure 

![matchExpressions](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/volumn_advanced.png)

## emptyDir 

- Pod 안에서 생성되므로 Pod가 문제가 될 경우 데이터가 Pod가 없어질 때 사라질 수 있음 
  - 언제 삭제되어도 상관이 없는 데이터를 담아야함! 

```yaml
apiVersion: v1
kind: Pod 
metadata:
  name: pod-volume-1 
spec:
  containers:
  - name : container1 
    image : tmkube/init 
    volumeMounts: 
      - name: empty-dir 
        mountPath: /mount1 
  - name : container2 
    image: tmkube/init 
    volumeMounts: 
    - name : empty-dir 
      mountPath : /mount2 
  volumns: 
  - name: emtpy-dir
    emptyDir: {}
```

## hostPath

- 각 Node 의 path 사용  
  - hostPath 는 자기 Node 내에서만 사용이 가능함.
  - 각각의 노드에는 각 노드 자신을 위해서 사용되는 파일들. Pod 내의 본인 호스트의 파일 내용 읽거나 써야함. 
  - Host Path 는 Node 에 있는 데이터를 파드에서 사용하기 위함임.
  - Pod 간의 파일 공유가 가능함!!

```yaml
apiVersion: v1
kind: Pod 
metadata: 
  name: pod-volume-2 
spec:
  containers:
  - name: container 
    image: tmkube/init
    volumeMounts:
    - name: host-path 
      mountPath: /mount1 
  volumes: 
  - name: host-path 
    hostPath: 
      path: /node-v  # 사전에 반드시 Node에 경로가 있어야함! 
      type: DirectoryOrCreate
```

## PVC/PV

- Persistent Volume Claim 
  - Persistent Volume

### PV 정의 생성 

```yaml
apiVersion: v1
kind: PersistentVolume 
metadata: 
  name: pv-01 
spec: 
  capacity: # 중요
    storage: 1G 
  accessModes: # 중요 
    - ReadWriteOnce / ReadOnlyMany
  local:
    path: /node-v 
  nodeAffinity: 
    require: 
      nodeSelectorTerms: 
        - matchExpressions: 
          - {key: kubernetes.io/hostname , operator: ln, values: [k8s-node1]}
  
```

### PVC 생성

```yaml
apiVersion: v1 
kind: PersistentVolumeClaim 
metadata: 
  name: pvc-01
spec: 
  accessModes: 
    - ReadWriteOnce 
  resources: 
    requests: 
      storage: 1G
storageClassName: ""
```

```yaml
apiVersion: v1 
kind: PersistentVolumeClaim 
metadata: 
  name: pvc-01
spec: 
  accessModes: 
    - ReadOnlyMany  
  resources: 
    requests: 
      storage: 3G
storageClassName: ""
```

### PVC - PV 연결

### Pod 생성시 마운팅

```yaml
apiVersion: v1 
kind: Pod 
metadata: 
  name: pod-volume-3 
spec:
  containers: 
  - name: container 
    image: tmkube/init 
    volumeMounts: 
    - name: pvc-pv
      persistentVolumeClaim:
        claimName: pvc-01         
```

# Object - ConfigMap, Select

- 각 서비스의 설정에 따라서 개발 / 상용 환경에 따라서 설정을 다르게 구성할 수 있어야함. 
  - SSH, User, Key etc 
- 일반 상수를 모아서 관리할 수 있는 것을 말하며 Pod 기동시 각각의 설정을 일거가서 사용할 수 있음.  
  - ConfigMap : 일반 상수 관련 
  - Secret : 보안 관련 
- 컨테이너 생성시 설정항목을 환경 변수처럼 설정할 수 있도록 세팅해두면 Config Map 만을 변경하여 개발, 상용으로 분류 가능함. 

## ConfigMap 

- Key : Value

## Secret 

데이터는 메모리에 저장되어 있음, 1 메가 바이트까지만 저장할 수 있음. 
시크릿이 너무 많아지면 메모리 상의 이슈가 될 수 있음 

- Key : Value 
  - Base64로 구성해야함!!

## Sample 

```yaml
apiVersion: v1 
kind: Pod 
metadata:
  name: pod-1
spec: 
  containers: 
    - name : container 
      image : tmkube/init 
      evnFrom : 
        - configMapRef:
            name: cm-dev # Config Map 을 연동 
        - secretRef: 
            name: sec-dev # Secret 을 연동 
```

```yaml
apiVersion: v1 
kind: configMap 
metadata: 
  name: cm-dev 
data: 
  SSH: False 
  User: dev
```

```yaml
apiVersion: v1 
kind: Secret 
metadata: 
  name: sec-dev 
data: 
  Key: MTlzNA==
```

## Env ( File )

환경 변수 방식은 한번 주입하면 끝이므로 파드의 환경변수에 대한 변경은 불가함. 파드가 다시 만들어질 때 가능함. 

```shell

# Config Map 생성
# Base 64로 변경되기 때문에 내부 파일에 Base64로 되어 있으면 두번 암호화가 되는 것임!   
$ kubectl create configmap cm-file --from-file=./file.txt 

# 
$ kubectl create secret generic sec-file --from-file=./file.txt 

```

```yaml
apiVersion: v1 
kind: Pod 
metadata: 
  name: file 
spec: 
  containers: 
    - name : container 
      image : tmkube/init 
      env: 
        - name: file 
          valueFrom: 
            configMapKeyRef: 
              name: cm-file 
              key: file.txt 
```

## Volume Mount(File)

볼륨의 경우 볼륨에서 수정한 설정의 경우 동작중인 파드에 영향을 미치고 있음 

```yaml
apiVersion: v1 
kind: Pod 
metadata: 
  name: mount 
spec: 
  containers: 
    - name: container 
      image: tmkube/init 
      volumeMounts: 
      - name: file-volume 
        mountPath : /mount 
volumes: 
  - name: file-volume 
    configMap: 
      name: cm-file 
```

## Dynamic Provisioning

![Namespace, Resource Quota, Limit Range](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/dynamic_provisioning.png)

- StorageClass  

- Status & ReclaimPolicy 

# Object - Namespace, ResourceQuota, LimitRange

![Namespace, Resource Quota, Limit Range](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/kubenetes_image01.png)

## Namespace 

- Pod 명이 중복될 수 없음. Namespace 각각에서는 동일한 파드명을 사용할 수 있다. 
- 각각의 서로 다른 Namespace 간에서 Pod는 서로 네트워크 연결이 가능하지만 이 제어는 Namespace Policy를 이용할 수 있다. 

### Namespace 1 

```yaml
apiVersion: v1 
kind: Namespace 
metadata: 
  name: nm-1 
```

```yaml
apiVersion: v1 
kind: Pod 
metadata: 
  name: pod-1 
  namespace: nm-1 
  labels: 
    nm : pod1 
spec: 
  containers:
  ... 
```

### Namespace 2 

```yaml
apiVersion: v1 
kind: Namespace 
metadata: 
  name: nm-2 
```

```yaml
apiVersion: v1 
kind: Pod 
metadata: 
  name: pod-1 
  namespace: nm-2 
  labels: 
    nm : pod1 
spec: 
  containers:
  ... 
```

## Resource Quota 

```yaml
apiVersion: v1 
kind: ResourceQuota 
metadata:
  name: rq-1 
  namespace: nm-1 
spec: 
  hard: 
    requests.memory: 3Gi 
    limits.memory: 6Gi
```

```yaml
apiVersion: v1 
kind: Pod 
metadata:
  name: pod-2 
spec:
  containers: 
  - name: container 
    image: tmkube/app
  resources:
    requests:
      memory: 2Gi 
    limits:
      memory: 2Gi 
```

## Limit Range 

```yaml
apiVersion: v1 
kind: LimitRange 
metadata: 
  name: lr-1
  namespace: nm-1 
spec:
  limits:
    - type: Container 
      min: 
        memory: 1Gi 
      max: 
        memory: 4Gi 
      defaultRequest: 
        memory: 1Gi 
      default: 
        memory: 2Gi 
      maxLimitRequestRatio: 
        memory: 3
```

# Object - Controller 

- Auto Healing 
- Auto Scaling
- Job
  - CronJb
  - Job
- Upgrade/Rollback 

## Controller - Replication Controller, ReplicaSet 

- Template

```yaml
apiVersion: v1
kind: Pod 
metadata: 
  name: pod-1 
  labels: 
    type: web
spec:
  containers:
    - name: container 
      image: tmkube/app:v1
```

```yaml
apiVersion: v1 
kind: ReplicationContoller 
metadata: 
  name: replication-1 
spec: 
  replicas: 1
  selector: 
    type: web 
  template: 
    metadata: 
      name: pod-1 
      labels: 
        type: web 
    spec: 
      containers: 
      - name: container 
        image: tmkube/app:v2
```

- Replicas

- Selector
  - Replicas에서만 존재하는 기능임. 
  - matchLabels 
  - matchExpressions

![matchExpressions](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/matchExpressions.png)

```yaml
apiVersion: v1 
kind: ReplicaSet 
metadata: 
  name: replica-1 
spec:
  replicas: 3
  selector: 
    matchLabels: 
      type: web 
    matchExpressions: 
    - {key: ver, operator: Exists}
    # Exists, DoesNotExist, In, Notin
    # Exists : 내가 키를 정하고 내가 가지고 있는 키를 가지고 파드를 가져와서 들어감. 
    # DoesNotExist
    # In : 키와 값을 지정할 때, 포함되는 것들을 모두 가져옴. 
    # NotIn: 해당되지 않는 파드 가져올 수 있게 처리 
  template: 
    metadata: 
      name: pod 
```

## Controller - Deployment 

![Deployments](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/deployments.png)

- Recreate 
  - 단점: 다운 타임이 발생하므로 일시적으로 서비스 중단이 가능한 경우에만 사용 가능함. 
- Rolling Update 
  - v1, v2를 모두 배포해둔 상태에서 v1의 pod 를 삭제하고 신규 v2 파드를 만드는 방식으로 단계적으로 변경 처리됨. 
  - 배포 과정 상에 추가적인 리소스를 필요로함. 
- Blue/Green 
  - Controller 를 이용해서 만들어진 Pod 에 대해서 추가적으로 v2 버전의 컨트롤러를 만들고 서비스의 있는 라벨을 변경하여 v2 버전으로 바로 연결하여 사용 하는 방식 
  - 서비스에 대한 다운 타임은 없고, 문제가 발생할 경우 서비스의 label 명만 바꿔주게 되면 v1을 다시 사용하는 방식임
  - 이 경우에는 리소스는 2배로 일시적으로 유지될 수 있음. 
- Canary 
  - 실험체를 통해서 위험을 검증하고 위험이 없다면 정식으로 배포하는 것 
  - 기존에 운영중인 v1에 대해서 연결된 상태에서 서비스로 v2를 추가 연결하여 새버전에 대한 테스트를 진행한다. ( 불특정 다수의 테ㅅ트)
  - 또는 각각의 서비스를 만들어서 잉그레스를 통한 서비스에 연결해서 사용하는 방식으로 사용하게 됨. 
    - 잉그레스 컨트롤러를 통해서 v1, v2 url path 에 따라서 분배가 가능하게 되는 구조임. 

### ReCreate 

```yaml
apiVersion: v1 
kind: Service 
metadata: 
  name: svc-1 
spec: 
  selector: 
    type: app 
  ports: 
    - port: 8080
      protocol: TCP 
      targetPort: 8080
```

```yaml
apiVersion: apps/v1 
kind: Deployment 
metadata: 
  name: deployment-1 
spec: 
  selector: 
    matchLabels: 
      type: app 
  replicas: 2
  strategy: 
    type: Recreate 
  revisionHistoryLimit: 1 # ReplicaSet 의 갯수를 제한할 수 있도록 수량 관리 
  template: 
    metadata: 
      labels: 
        type: app 
    spec: 
      containers: 
      - name: container 
        image: tmkube/app:v1 
```

### Rolling Update

```yaml
apiVersion: v1 
kind: Service 
metadata: 
  name: svc-1 
spec: 
  selector: 
    type: app 
  ports: 
    - port: 8080
      protocol: TCP 
      targetPort: 8080
```

```yaml
apiVersion: apps/v1 
kind: Deployment 
metadata: 
  name: deployment-1 
spec: 
  selector: 
    matchLabels: 
      type: app 
  replicas: 2
  strategy: 
    type: RollingUpdate
  minReadySeconds: 10  
  template: 
    metadata: 
      labels: 
        type: app 
    spec: 
      containers: 
      - name: container 
        image: tmkube/app:v1 
```
## Controller - DaemonSet, Job, CronJob 

### DaemonSet

노드의 자원 상태와 상관 없이 모든 노드에 파드가 하나씩 생긴다는 특징이 있음 

- 주로 성능 수집/상태 관리 
- 프로메테우스 와 같은 성능 수집 파드 설정 ( Prometheus ) 
- 문제 파악을 위한 로그 ( Fluentd )
- 각각의 자원을 세팅 함. ( GlusterFS )

```yaml
apiVersion: v1 
kind: DaemonSet 
metadata: 
  name: daemonset-1 
spec: 
  selector: 
    matchLabels: 
      type: app
  template: 
    metadata: 
      labels: 
        type:app 
    spec:
      nodeSelector:
        os: centos 
      containers: 
        - name: container
          image: tmkube/app 
          paths: 
          - containerPort: 8080 
            hostPort: 18080 
```

### Job 

- Job을 통해 만들어진 Pod 가 있음 
- Controller 에 의해서 만들어진 Pod 들은 장애가 생겼을 때 복구가 가능할 수 있음
- ReplicaSet
  - Recreate
  - Restart

```yaml
apiVersion: batch/v1 
kind: Job 
metadata: 
  name: job-1 
spec: 
  completion: 6
  parallelism: 2
  activeDeadlineSeconds: 30 
  template: 
      spec: 
        restartPolicy: Never 
        containers: 
        - name:  container 
          image: tmkube/init
```

### CronJob 

- CronJob
  - Job들을 주기적으로 동작하게끔 사용함.
  - 아래와 같은 활용 범위가 있음 
    - Backup 
    - Checking 
    - Messaging

#### Concurrent Policy 

- Allow 
- Forbid 
- Replace

```yaml
apiVersion: batch/v1 
kind: CronJob 
metadata: 
  name: cron-job 
spec: 
  schedule: "*/1 * * * *"
  concurrencyPolicy: Allow 
  jobTemplate: 
    spec: 
      template:
        spec: 
          restartPolicy: Never 
          containers: 
          - name: container
            image: tmkube/app 
```

# Access to the Kubernetes API 

![Access Kubernetes API](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/access_kubernetes_api.png)

## Authentication

### X509 Certs, kubectl, ServiceAccount 

![Access Kubernetes API](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/authentication.png)

### RBAC, Role, RoleBinding Detail 

![Access Kubernetes API](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/authrization001.png)

# Kubernetes Dashboard 구성 

- ClusterRole 
  - ClusterRoleBinding 

- kubeconfig 
  - Client crt
  - Client Key 

- kubernetes-metrics-scraper 

# StatefulSet 

![Stateful Set](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/statefulset.png)

## Stateless Application

- 볼륨이 반드시 필요하지 않음. 
- 요청에 대해서 분산이 될 수 있도록 처리 

### ReplicaSet

## Stateful Application

- 각각의 역할에 따라서 Volume 각각 나뉘어지는 경우가 많음
- 각 App의 역할에 맞게 요청이 수신이 들어와야함, Stateless와 분산의 개념이 다름. 

### StatefulSet 

- Ordinal Index 로 순차적으로 Pod가 생성됨. 
  - 중간에 Pod가 삭제가되면 동일한 이름으로 인덱스가 세팅이 됩니다.

# Ingress 

![Stateful Set](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/ingress.png)

![Stateful Set](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/ingress_controller.png)

- 서비스 로드 밸런싱 
  - 각 서비스의 로드 밸런싱을 담당함. 각각의 Pod에 연결된 서비스에 맞게 보내주는 역할을 함. 
- Canary Upgrade 
  - 서비스 배포시 버전1의 앱과 버전2의 앱 사이의 배포시에 트래픽을 잉그레스를 통해서 조정한다. 

## Ingress Controller 

- Nginx, Kong 

# Autoscaler - HPA 

![HPA](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/hpa.png)

- HPA 
  - HPA의 리소스를 체크하고 있다가 컨트롤러의 ReplicaSet를 증가시키는 작업을 할 수 있다. 
    - Scale Out 
    - Scale In 
  - 기동이 빠르게 되는 App 
  - Stateless App  

![HPA Detail](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/hpa_detail.png)

- VPA
  - StateFul App에 대해서 수직적으로 리소스를 증가시키는 작업을 수행함. 
    - Scale Up 
    - Scale Down 
- CA 
  - 동적으로 워커노드를 추가시킬 경우, Node를 추가하여 Pod를 이동하여 동작할 수 있도록 하는 것! 

## HPA 