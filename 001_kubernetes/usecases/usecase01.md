# Deployment 에 대해서 Kubectl CLI 를 이용핸 생성 방식에 대해서 

- [Kubectl Create Command](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-deployment-em-)

```shell 
$ kubectl create deployment NAME --image=image -- [COMMAND] [args...]
```

```shell 
$ kubectl create deployment dep-2-replicas --image=busybox --replicas=2
```

```yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: busybox-deployment
  labels:
    app: busybox
spec:
  replicas: 2
  selector:
    matchLabels:
      app: busybox
  template:
    metadata:
      labels:
        app: busybox
    spec:
      containers:
      - name: busybox
        image: busybox
```

# Deployment를 Record 하면서 Image 를 교체를 Kubectl CLI를 이용해서 실행하는 경우 

- [Kubectl Create Command](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-deployment-em-)

```shell 
$ kubectl create deployment NAME --image=image -- [COMMAND] [args...]
```

```shell
$ kubectl create deployment nginx-deploy --image=nginx:1.16 --replicas=1 --record=true 

# One Way
$ kubectl set image deployment.v1.apps/nginx-deploy nginx=nginx:1.17

$ kubectl rollout history deployment nginx-deploy 

# Second Way 
$ kubectl edit deployment/nginx-deploy 

# Apply vim with replace with .spec.template.spec.containers[0].image

# history checking 
$ kubectl rollout history deployment nginx-deploy 

# Confirm record is worked 
$ kubectl describe deployment nginx-deploy 
```