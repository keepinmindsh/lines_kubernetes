# Kubernetes - Hello World 

- [Link 바로가기](https://github.com/keepinmindsh/lines_kubernetes/tree/main/005_BasicKubernetes/001)

# Kubernetes의 이해

- [Link 바로가기](https://github.com/keepinmindsh/lines_kubernetes/tree/main/005_BasicKubernetes/002)

# Kubernetes - Objects 

- [Pod 에 대한 Link 바로가기](https://github.com/keepinmindsh/lines_kubernetes/tree/main/005_BasicKubernetes/003)
- [Service 에 대한 Link 바로가기](https://github.com/keepinmindsh/lines_kubernetes/tree/main/005_BasicKubernetes/004)
- [Volume 에 대한 Link 바로가기](https://github.com/keepinmindsh/lines_kubernetes/tree/main/005_BasicKubernetes/005)

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

--- 

- Pod 명이 중복될 수 없음. Namespace 각각에서는 동일한 파드명을 사용할 수 있다. 
- 각각의 서로 다른 Namespace 간에서 Pod는 서로 네트워크 연결이 가능하지만 이 제어는 Namespace Policy를 이용할 수 있다. 

---

Service Accounts는 쿠버네티스 API에 의해서 관리되는 사용들이다. 각 계정은 특정한 namespaces에 연결된다. Service Account는 Credentials의 묶음으로 Secrets 로서 저장된다. 
그리고 클러스터 내에서 쿠버네티스 API와 통신/대화하는 것을 허용하는 각각의 Pod들로 마운트된다. 

---

> [Name Spaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)  
> [Name Spaces by resources quota](https://kubernetes.io/docs/concepts/policy/resource-quotas/)  
> [Share a Cluster with Namespaces](https://kubernetes.io/docs/tasks/administer-cluster/namespaces/)  
> [Namespace와 ServiceAccount의 상관관계 설명](https://kubernetes.io/docs/reference/access-authn-authz/authentication/)


### Namespace 생성 

현재의 Namespace에서 특정 서비스 어카운트로 생성되는데 아래의 코드와 같이 동작한다. 

- Service Account 생성 예제 

```shell 

kubectl create serviceaccount jenkins

```

- Token 생성 예제 

```shell 

kubectl create token jenkins

```


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


##### Service Account 

- kubernetes의 service account 생성하기

```shell 

kubectl apply -f - <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: build-robot
EOF

```

- Service Account 명 생성 규칙 

**DNS Subdomain Names**

Most resource types require a name that can be used as a DNS subdomain name as defined in [RFC 1123](https://tools.ietf.org/html/rfc1123). This means the name must:

1. contain no more than 253 characters.     
2. contain only lowercase alphanumeric characters, '-' or '.'     
3. start with an alphanumeric character.    
4. end with an alphanumeric character.  

- 실제 생성된 계정을 확인해보면, 

```shell

kubectl get serviceaccounts/build-robot -o yaml

```

- Token를 생성하려면, ( Kubectl - Dashboard  로그인 시에도 사용 가능 )

```shell 

kubectl create token build-robot

```

- 오랫동안 유지될 수 잇는 API Token을 만들어야 하는 경우 

```shell 

kubectl apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: build-robot-secret
  annotations:
    kubernetes.io/service-account.name: build-robot
type: kubernetes.io/service-account-token
EOF

```

- token 정보를 확인하려면,  

```shell 

kubectl get secret/build-robot-secret -o yaml

```

- secret 정보 설명을 확인하려면, 

```shell 

kubectl describe secrets/build-robot-secret

```

- 실제 정보를 조회한 결과 

```shell 

kubectl describe secrets/default-secret
Name:         default-secret
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: default
              kubernetes.io/service-account.uid: 629a131a-6adc-460a-b61a-9207deead3d2

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1099 bytes
namespace:  7 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IjZBNU5HVnlKX1owaWU4NG9wTktVSldqQ0p5U241a0FocFNUWFV0cE9majgifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRlZmF1bHQtc2VjcmV0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImRlZmF1bHQiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI2MjlhMTMxYS02YWRjLTQ2MGEtYjYxYS05MjA3ZGVlYWQzZDIiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZGVmYXVsdDpkZWZhdWx0In0.YBFz3B4UgCQHJUyKSxDagFjGKLqirBOGjUCRo_7YRznxGP982ldYf6_-UeTkCoDA8sFZ7YiL5GJICOqmb1Hut2Qi6casvURfHs0LmAPVxefF5-WRcOpkiT-KLjuBE7vhBlgU_sXwE4bHIsxeS0afYCydDRHxRQj0QkCEnTK2UEzHP0S3yLzNHGQscYU9CEv5ZzggxSaL7xf9In2RdazezDlhUw2R9PMjXM6pMDyPK6-gLIV-XZSuhnc11CNboYQdeKvzy7sLLddwxvBlhPurqFtwVeEfnSbv2SmuSApneYsdGk6362rcvDbM0pHK4xebMmBpaa487wIU62q6QMlZqQ

```


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
