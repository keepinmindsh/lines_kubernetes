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