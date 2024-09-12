# Exam 1 

```shell
$ kubectl get nodes
NAME                                             STATUS   ROLES    AGE   VERSION
gke-oscar-cluster-2-default-pool-a533dc83-ew5k   Ready    <none>   26d   v1.26.5-gke.1200
gke-oscar-cluster-2-default-pool-b3cbdcb2-9wdi   Ready    <none>   26d   v1.26.5-gke.1200
gke-oscar-cluster-2-default-pool-c68e7d97-44o9   Ready    <none>   26d   v1.26.5-gke.1200

howard /Users/data/import_temp/import
 
$ kubectl get nodes > howard-node.json

$ vi howard-node.json
NAME                                             STATUS   ROLES    AGE   VERSION
gke-oscar-cluster-2-default-pool-a533dc83-ew5k   Ready    <none>   26d   v1.26.5-gke.1200
gke-oscar-cluster-2-default-pool-b3cbdcb2-9wdi   Ready    <none>   26d   v1.26.5-gke.1200
gke-oscar-cluster-2-default-pool-c68e7d97-44o9   Ready    <none>   26d   v1.26.5-gke.1200

```


# Exam 2 

```shell
$ kubectl create namespace NAME [--dry-run=server|client|none]
```

```shell
kubectl create namespace exam-jsh
```

# Exam 3

```shell 
$ kubectl create deployment NAME --image=image -- [COMMAND] [args...]
```

```shell 
$ kubectl create deployment  nginx-jsh --image=nginx --replicas=2 --namespace=exam-jsh
deployment.apps/nginx-jsh created

$ kubectl get deployment -n exam-jsh
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
nginx-jsh   2/2     2            2           70s
```

# Exam 4

우선 현재 기동되고 있는 Deployment 의 yaml 정보를 확인한다. 

```shell
# Deployments 목록 전체 조회 
$ kubectl get deployments -o yaml

# 특정 Deployment 단건 조회 
$ kubectl get deployments oscar-server-2 -o yaml 
```

```shell
 kubectl get [(-o|--output=)json|yaml|name|go-template|go-template-file|template|templatefile|jsonpath|jsonpath-as-json|jsonpath-file|custom-columns|custom-columns-file|wide] (TYPE[.VERSION][.GROUP] [NAME | -l label] | TYPE[.VERSION][.GROUP]/NAME ...) [flags]
```

```shell 
$ kubectl get deployments oscar-server-2 -o custom-columns='DEPLOYMENT:.metadata.name','CONTAINER_IMAGE:.spec.template.spec.containers[*].image','READY_REPLICA:.status.readyReplicas','NAMESPACE:.metadata.namespace'
DEPLOYMENT       CONTAINER_IMAGE                                                                                       READY_REPLICA   NAMESPACE
oscar-server-2   gcr.io/swit-dev/oscar-image@sha256:5c6b0247ac0ea4135d2b27a28689b533208518d917328482c60cb2ad1e703c19   3               default
```

# Exam 5


```shell
$ kubectl create deployment nginx-deploy --image=nginx:1.16 --namespace=exam-jsh --replicas=1

$ kubectl edit deployment nginx-deploy -o yaml --save-config -n exam-jsh

$ kubectl rollout history deployment/nginx-deploy -n exam-jsh
deployment.apps/nginx-deploy
REVISION  CHANGE-CAUSE
1         <none>
2         <none>

```

# Exam 6


```yaml 
apiVersion: v1
kind: Service
metadata:
  name: web-application
spec:
  type: NodePort
  selector:
    app.kubernetes.io/name: simple-webapp
  ports:
      # By default and for convenience, the `targetPort` is set to the same value as the `port` field.
    - port: 8080
      targetPort: 8080
      # Optional field
      # By default and for convenience, the Kubernetes control plane will allocate a port from a range (default: 30000-32767)
      nodePort: 30083
```


```shell
$ kubectl create -f service.yaml -n exam-jsh

$ kubectl get service -n exam-jsh

NAME              TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
web-application   NodePort   10.36.15.77   <none>        8080:30083/TCP   60s
```


# Exam 7 

```shell
$ kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-jsh
spec:
  selector:
    matchLabels:
      run: hello-jsh-example
  replicas: 2
  template:
    metadata:
      labels:
        run: hello-jsh-example
    spec:
      containers:
        - name: hello-jsh
          image: gcr.io/google-samples/node-hello:1.0
          ports:
            - containerPort: 8080
              protocol: TCP
EOF

$ deployment.apps/hello-jsh created
```

```shell
$ kubectl expose deployment hello-jsh --type=NodePort --name=jsh-service
$ service/jsh-service exposed
```

# Exam 8 

```shell
$ kubectl create deployment jsh-deployment --image=nginx:1.16 --replicas=2
deployment.apps/jsh-deployment created

$ kubectl scale --replicas=1 deployment/jsh-deployment -n exam-jsh
deployment.apps/jsh-deployment scaled

$ kubectl rollout history deployment/jsh-deployment
deployment.apps/jsh-deployment
REVISION  CHANGE-CAUSE
1         <none>

$ kubectl apply -f - <<EOF 
apiVersion: v1
kind: ConfigMap
metadata:
  name: hello-config
data: 
  hello.properties: |
    hello=world
    drink=good
    happy=work
EOF 
```

```shell
$ kubectl edit deployment jsh-deployment
```


```yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2023-08-10T12:58:16Z"
  generation: 2
  labels:
    app: jsh-deployment
  name: jsh-deployment
  namespace: default
  resourceVersion: "61660304"
  uid: 6f86f7c5-2695-48ff-b2ff-bd50b6dcd3b0
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: jsh-deployment
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: jsh-deployment
    spec:
      containers:
      - image: nginx:1.16
        imagePullPolicy: IfNotPresent
        name: nginx
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts: 
        - name: config 
          mountPath: "/config" 
          readOnly: true 
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      volumes:
      # 파드 레벨에서 볼륨을 설정한 다음, 해당 파드 내의 컨테이너에 마운트한다.
      - name: config
        configMap:
          # 마운트하려는 컨피그맵의 이름을 제공한다.
          name: game-demo
          items: 
          - key: "hello.properties"
            path: "hello.properties"
```

# Exam 9 

```shell
kubectl exec nginx-deploy-6b66b677ff-t8m2p -n exam-jsh  -it -- bin/sh

# echo "helloworld" > hello.txt

# ls -al | grep hello
-rw-r--r--   1 root root   11 Aug  3 09:40 hello.txt
```

# Exam 10 

> [Init Container](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)


```shell 
$ kubectl create -f pod.yaml -n exam-jsh

$ kubectl get pod -n exam-jsh
NAME                              READY   STATUS     RESTARTS   AGE
jsh-deployment-55b5cfd494-xcqbh   1/1     Running    0          11m
myapp-pod                         0/1     Init:0/1   0          52s
nginx-deploy-6b66b677ff-dw5xk     1/1     Running    0          27m
nginx-deploy-6b66b677ff-t8m2p     1/1     Running    0          27m
nginx-jsh-675c95594c-69fl7        1/1     Running    0          44m
nginx-jsh-675c95594c-7tthf        1/1     Running    0          44m
```

```yaml 
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app.kubernetes.io/name: MyApp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
```