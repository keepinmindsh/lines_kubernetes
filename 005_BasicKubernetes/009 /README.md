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
    - Replicas 에서만 존재하는 기능임.
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

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # modify replicas according to your case
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
```

- Replica Set 의 설정 가져오기 ( Shell )

```shell
$ kubectl get rs
NAME                                      DESIRED   CURRENT   READY   AGE
lines-admin-nextjs-deployment-dd954f84b   1         1         1       2d19h
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