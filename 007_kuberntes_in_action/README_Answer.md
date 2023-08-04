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

```shell
 kubectl get [(-o|--output=)json|yaml|name|go-template|go-template-file|template|templatefile|jsonpath|jsonpath-as-json|jsonpath-file|custom-columns|custom-columns-file|wide] (TYPE[.VERSION][.GROUP] [NAME | -l label] | TYPE[.VERSION][.GROUP]/NAME ...) [flags]
```

```shell
$ kubectl get namespace
NAME              STATUS   AGE
default           Active   63d
exam-daisy        Active   2m57s
exam-dante        Active   2m59s
exam-jsh          Active   4m37s
exam-ryan         Active   3m28s
exam-sean         Active   2m6s
kube-node-lease   Active   63d
kube-public       Active   63d
kube-system       Active   63d
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
      nodePort: 30084
```


```shell
kubectl create -f service.yaml -n exam-jsh


kubectl get service -n exam-jsh

NAME              TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
web-application   NodePort   10.36.15.77   <none>        8080:30084/TCP   60s
```


# Exam 7 

```shell

kubectl create service clusterip jsh-service  --clusterip="None" -n exam-jsh


kubectl expose service jsh-service --port=8080 --target-port=9191 -n exam-jsh

```

# Exam 8 

```shell
kubectl create deployment jsh-deployment --image=nginx:1.16 --namespace=exam-jsh --replicas=2
deployment.apps/jsh-deployment created

kubectl scale --replicas=1 deployment/jsh-deployment -n exam-jsh
deployment.apps/jsh-deployment scaled



```


# Exam 9 


```shell
kubectl exec nginx-deploy-6b66b677ff-t8m2p -n exam-jsh  -it -- bin/sh

#
# echo "helloworld" > hello.txt

# ls -al | grep hello
-rw-r--r--   1 root root   11 Aug  3 09:40 hello.txt
```

# Exam 10 


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