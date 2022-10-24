# Pod

# Object - Service 

## ClusterIP 

- 클러스터내 접근 가능! 
  - 인가된 사용자 ( 운영자 )
  - 내부 대쉬보드 
  - Pod의 서비스 상태 디버깅 

```yaml
apiVerison: v1
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

# Object - Volume

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