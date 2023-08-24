# Exam 1

```
kubectl config set-context --current --namespace=howard-exam

kubectl run nignx --image=nginx --env=var1=var1 --namespace=howard-exam

kubectl get pods -n howard-exam
NAME    READY   STATUS    RESTARTS   AGE
nignx   1/1     Running   0          79s

kubectl exec nignx --namespace howard-exam  -it -- /bin/sh
```


# Exam 2 

```

kubectl run nginx --image nginx  -it -- bin/sh
If you don't see a command prompt, try pressing enter.

# echo "hello world" > main.txt
# ls
bin  boot  dev	docker-entrypoint.d  docker-entrypoint.sh  etc	home  lib  lib32  lib64  libx32  main.txt  media  mnt  opt  proc  root	run  sbin  srv	sys  tmp  usr  var
# ls -al
total 68
drwxr-xr-x   1 root root 4096 Aug 24 09:00 .
drwxr-xr-x   1 root root 4096 Aug 24 09:00 ..
lrwxrwxrwx   1 root root    7 Aug 14 00:00 bin -> usr/bin
drwxr-xr-x   2 root root 4096 Jul 14 16:00 boot
drwxr-xr-x   5 root root  380 Aug 24 09:00 dev
drwxr-xr-x   1 root root 4096 Aug 16 09:50 docker-entrypoint.d
-rwxrwxr-x   1 root root 1620 Aug 16 09:50 docker-entrypoint.sh
drwxr-xr-x   1 root root 4096 Aug 24 09:00 etc
drwxr-xr-x   2 root root 4096 Jul 14 16:00 home
lrwxrwxrwx   1 root root    7 Aug 14 00:00 lib -> usr/lib
lrwxrwxrwx   1 root root    9 Aug 14 00:00 lib32 -> usr/lib32
lrwxrwxrwx   1 root root    9 Aug 14 00:00 lib64 -> usr/lib64
lrwxrwxrwx   1 root root   10 Aug 14 00:00 libx32 -> usr/libx32
-rw-r--r--   1 root root   12 Aug 24 09:00 main.txt

```

# Exam 3 

```
kubectl run busybox --image=busybox -- bin/sh -c "while true; do echo 'Hi I am from Main container' >> /var/log/index.html; sleep 5; done"
pod/busybox created


kubectl logs busybox
bin/sh: can't create /var/log/index.html: nonexistent directory
bin/sh: can't create /var/log/index.html: nonexistent directory
bin/sh: can't create /var/log/index.html: nonexistent directory

```

# Exam 5 

```

kubectl run busybox-3 --image=busybox --command -- bin/sh -c "ls; sleep 3600;"  "echo Hello World; sleep 3600;" "echo this is the third container; sleep 3600"
```


# Exam 4


```
kubectl logs busybox-3

bin
dev
etc
home
lib
lib64
proc
root
sys
tmp
usr
var

```


# Exam 6

```
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: redis
    name: test-container
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir:
      sizeLimit: 500Mi



kubectl get pods
NAME        READY   STATUS    RESTARTS   AGE
busybox     1/1     Running   0          3m
busybox-3   1/1     Running   0          4m14s
test-pd     1/1     Running   0          8s
```

# Exam 7

```

kubectl run change-test --image=redis

kubectl edit pods change-test

kubectl get pods change-test -o yaml

status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2023-08-24T09:25:14Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2023-08-24T09:26:20Z"
    message: 'containers with unready status: [change-test]'
    reason: ContainersNotReady
    status: "False"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2023-08-24T09:26:20Z"
    message: 'containers with unready status: [change-test]'
    reason: ContainersNotReady
    status: "False"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2023-08-24T09:25:14Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: containerd://a74e2bb04c209fdb44b052c7a43e510242b4be83a07cfe56027af4b6cfdfcdcc
    image: docker.io/library/redis:latest
    imageID: docker.io/library/redis@sha256:c45b9ac48fde5e7ffc59e785719165511b1327151c392c891c2f552a83446847
    lastState:
      terminated:
        containerID: containerd://a74e2bb04c209fdb44b052c7a43e510242b4be83a07cfe56027af4b6cfdfcdcc
        exitCode: 0
        finishedAt: "2023-08-24T09:26:19Z"
        reason: Completed
        startedAt: "2023-08-24T09:25:16Z"
    name: change-test
    ready: false
    restartCount: 0
    started: false
    state:
      waiting:
        message: 'rpc error: code = Unknown desc = failed to pull and unpack image
          "docker.io/library/redis-1.17.1:latest": failed to resolve reference "docker.io/library/redis-1.17.1:latest":
          pull access denied, repository does not exist or may require authorization:
          server message: insufficient_scope: authorization failed'
        reason: ErrImagePull
  hostIP: 10.128.15.228
  phase: Running
  podIP: 10.32.8.125
  podIPs:
  - ip: 10.32.8.125
  qosClass: BestEffort
  startTime: "2023-08-24T09:25:14Z"

```

# Exam 8 


```
kubectl run change-alpine --image=1.11-alpine

kubectl edit pods change-alpine 

kubectl get pods chnage-alpine -o yaml 

```

# Exam 9 

```
kubectl run nginx --image nginx


pod/nginx created

kubectl expose pod  nginx --port=80 --target-port=6379
service/nginx exposed

```

# Exam 10 


```
kubectl run change-test --image=redis
pod/change-test created

kubectl expose pod change-test --port=6379 --target-port=6379
service/change-test exposed

```

# Exam 11 


```
kubectl delete pod change-test --now
pod "change-test" deleted

```

# Exam 12 


```
  kubectl run nginx-dev --image=nginx
pod/nginx-dev created
 howard  ~/sources/01_company_git
  kubectl run nginx-prod --image=nginx
pod/nginx-prod created
 howard  ~/sources/01_company_git
 kubectl delete pod nginx-dev nginx-prod
pod "nginx-dev" deleted
pod "nginx-prod" deleted

```


# Exam 13 

```
kubectl get pods -o yaml

kubectl get pods  -o custom-columns=CONTAINER:.metadata.namespace,IMAGE:.metadata.
```


# Exam 14 


# Exam 15 


