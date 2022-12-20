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

GCP 의 경우 Cluster 및 Node 구성시 Storage 가 기본적으로 매핑됨. 

### ConfigMap, Secret 

### Namespace, ResourceQuota, LimitRange 

> [Trouble Shooting - Stack OverFlow](https://stackoverflow.com/questions/53452120/gcp-kubernetes-workload-does-not-have-minimum-availability)     
> [Trouble Shooting - GCP](https://cloud.google.com/kubernetes-engine/docs/troubleshooting)