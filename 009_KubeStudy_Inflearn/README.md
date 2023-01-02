# 1차 Study 자료 

### 사전 준비 

##### 자동완성으로 shell을 세팅하려면 아래와 같이 zsh | bash 에 설정 해야함.

```shell

# K8S Setting
alias k=kubectl
source <(kubectl completion zsh) ## 이게 추가 되어야 함! 

```

- zshrc
    - https://kubernetes.io/docs/tasks/tools/included/optional-kubectl-configs-zsh/
- bashrc
    - https://kubernetes.io/docs/tasks/tools/included/optional-kubectl-configs-bash-linux/

##### Docker Desktop 을 구성 

##### Docker Image 빌드 

```shell

$ docker build . -t lines_admin_nextjs -f /Users/lines/sources/01_bonggit/admin-site/frontend/deployments/Dockerfile

```

##### Docker Container 기동 / 정상 동작 확인 

```shell

$ docker run --rm -it -p 8080:3000 --name lines-admin-front lines_admin_nextjs:latest

```

```shell

$ docker push gcr.io/lines-infra/lines_admin_front:v1.0

```


### Pod 구성 

```shell

$ kubectl run lines-cluster --image=gcr.io/lines-infra/lines_admin_front:v1.0 --port=3000  

```

- Pod Lifecyle 구성  
  - Pending: Pod has been created and accepted by the cluster, but one or more of its containers are not yet running. 
    This phase includes time spent being scheduled on a node and downloading images.
  - Running: Pod has been bound to a node, and all of the containers have been created. 
    At least one container is running, is in the process of starting, or is restarting.
  - Succeeded: All containers in the Pod have terminated successfully. 
    Terminated Pods do not restart.
  - Failed: All containers in the Pod have terminated, and at least one container has terminated in failure. 
    A container "fails" if it exits with a non-zero status.
  - Unknown: The state of the Pod cannot be determined.

```shell

# https://cloud.google.com/code/docs/intellij/install 

# https://github.com/GoogleCloudPlatform/cloud-code-intellij/releases   - GoLand Plugins 
# - Cloud Code 를 설정하면 자유롭게 yaml 을 빌드해서 사용할 수 있다. 

# https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get 

# kubectl and configure cluster access 설정 
# https://cloud.google.com/kubernetes-engine/docs/how-to/cluster-access-for-kubectl

```

```shell

$ gcloud container clusters list

```

```shell

$ kubectl apply -f helloworld.deployment.yaml --namespace=default

```

```shell

$ kubectl get pods

```

```shell

# https://linuxhandbook.com/kubectl-delete-deployment/ 
$ kubectl delete deployments lines-admin-nextjs-deployment

```

```shell

$ kubectl delete service lines-admin-front-http

```

```shell

$ kubectl

```

### Service 구성 

```shell

$ kubectl apply -f helloworld.service.yaml

# 변경 
$ kubectl expose pod lines-cluster --type=LoadBalancer --name lines-admin-front-http

```

### Volume 

GCP 의 경우 Cluster 및 Node 구성시 Storage 가 기본적으로 매핑되었음

### ConfigMap, Secret 

### Namespace, ResourceQuota, LimitRange 

> [Trouble Shooting - Stack OverFlow](https://stackoverflow.com/questions/53452120/gcp-kubernetes-workload-does-not-have-minimum-availability)     
> [Trouble Shooting - GCP](https://cloud.google.com/kubernetes-engine/docs/troubleshooting)

# 2차 스터디 자료 

### Controller 

- Auto Healing 
- Auto Scaling 
- Software Update 
- Job 

##### ReplicaSet / Selector 

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
  selector: # 키와 값이 같아야 설정할수 있게 처리, matchExpressions 에는 key, operator ( Exists, DoesNotExist, In, NotIn )를 설정할 수 있음  
    matchLabels:
      tier: frontend
  template: # 서비스가 장애가 나거나 정상적으로 동작하지 않을 때 Template 으로 사용이 가능하다. 
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
```

##### Deployment 

- ReCreate 
  - 다운 타임이 발생하므로 일시적으로 서비스 정지가 가능한 서비스에서만 사용 가능함. 
- Rolling Update 
  - 다운 타임 없으나, 리소스 비용이 일시적으로 증가함. 
- Blue/Green 
  - 서비스 다운 타임 없음. 리소스 비용이 2배가 필요함. 
- Canary                                 
  - 서비스 다운 타임 없음. 리소스 비용은 설정에 따라 증가하게됨. 

##### DaemonSet, Job, CronJob 

- DaemonSet
  - Node의 자원 상태와 상관 없이 각 Node에 Pod가 생성됨. 
    - 성능 수집 ( 모니터링 - 프로메테우스 ), 로그 수집(Fluentd), 스토리지 활용 (GlusterFS)

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      # these tolerations are to have the daemonset runnable on control plane nodes
      # remove them if your control plane nodes should not run pods
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```

- Job / CronJob 
  - Job 에 의해 만들어진 Pod 
    - 서비스 다운시 다시 다른 노드에 재생성되는 구조
    - 프로세스가 일을 하지 않으면 Pod는 정지됨. 
  - CronJob 
    - 주기적으로 Job을 실행하기 위해서 사용함. 
      - BackUp 
      - Checking
      - Messaging 
      - etc 

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  completions: 6  # Job이 6개가 종료되기 전까지 완료되지 않음 
  parallelism: 2  # 병렬적으로 Job을 실행함. 
  activeDeadlineSeconds: 30 # 30초 후에 Job이 삭제됨. 
  template:
    spec:
      containers:
      - name: pi
        image: perl:5.34.0
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4

```

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "* * * * *"
  concurrencyPolicy: Allow # Forbid, Replace 
  # Allow - 스케쥴에 따라서 Pod가 생성될 때 기존의 Pod의 종료 여부에 상관 없이 Pod가 스케쥴에 따라 무조건 생성됨. 
  # Forbid - 기존의 Pod가 종료되지 않았다면 스케쥴 시간이 왔을 때 Job을 건너띄고 이후 Pod가 종료된 시점 이후에 Job이 추가 실행됨. 
  # Replace - 기존의 Pod가 종료되지 않은 시점에 스케쥴 시간이 되면 Pod는 새로 만들어지지만 Job은 신규 Pod로 연결되는 구조가 된다. 
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

