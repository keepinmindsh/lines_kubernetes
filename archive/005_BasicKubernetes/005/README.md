
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