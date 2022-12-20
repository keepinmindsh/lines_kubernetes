# Kubernetes - Hello World

- 기동할 App.js 만들기
```javascript
consthttp = require('http')
constos = require('os')

console.log("Kubia server starting...")

consthandler =function(request, response) {
    console.log("Received request from " + request.connection.remoteAddress);
    response.writeHead(200)
    response.end("You've hit " + os.hostname() + "\n")
};

constwww = http.createServer(handler);
www.listen(8080)
```

- Dockerfile 만들기
```dockerfile
FROM node:7
ADD app.js/app.js

ENTRYPOINT["node", "app.js"]
```

- Docker Image 빌드 하기
```shell
docker build . -t lines_appjs -f ~/sources/02_linesgits/lines_kubernetes/008_dockerfiles/213_dockerfilesamples/Dockerfile
```

- Docker Container 기동하기
```shell
docker run --name lines-appjs-container -p 8080:8080 -d lines_appjs
```

- 컨테이너 내부 정보 세세히 살피기
```shell
 docker inspect lines-appjs-container
```

- 컨테이너 내부에서 셀을 실행하기
    - -i : 표준 입력을 오픈 상태로 유지한다. 셸을 명령어를 입력하기 위해 필요하다.
    - -t : 의사 터미널을 할당한다.
```shell
$ docker exec -it lines-appjs-container bash

root@dac3baead0a0:/# ls
app.js	bin  boot  dev	etc  home  lib	lib64  media  mnt  opt	proc  root  run  sbin  srv  sys  tmp  usr  var
```

- 내부에서 컨테이너 탐색

```shell
root@dac3baead0a0:/# ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.5  1.2 786924 100244 ?       Ssl  13:21   0:01 /usr/bin/qemu-x86_64 /usr/local/bin/node node app.js
root        12  0.0  0.1 169292 11356 pts/0    Ssl  13:22   0:00 /usr/bin/qemu-x86_64 /bin/bash bash
root        22  0.0  0.1 166168  9876 ?        Rl+  13:21   0:00 ps aux
```

> 이와 같이 실행 중인 컨테이너에 진입하는 것은 컨테이너에 실행 중인 애플리케이션을 디버깅 할 때 유용하다. 문제가 있을 때  
> 가장 먼저 해야할 것은 애플리케이션이 보고 있는 시스템의 실제 상태를 탐색하는 것이다. 애플리케이션이 자체의 고유한 파일 시스템을
> 보고 있을 뿐만 아니라 프로세스, 사용자, 호스트 이름, 네트워크 인터페이스도 고유한 것을 보고 있다는 사실을 명심해야 한다.

- 컨테이너 중지와 삭제

```shell

$ docker stop lines-appjs-container 
$ docker rm lines-appjs-container

```

- 이미지 레지스트리에 이미지 푸시

```shell
$ docker tag {tag_name} {images_name}

$ docker images | head 
REPOSITORY                                                           TAG                                                                          IMAGE ID       CREATED         SIZE
gcr.io/tekton-releases/github.com/tektoncd/pipeline/cmd/webhook      <none>                                                                       8c011a76d60c   9 days ago      74.7MB
gcr.io/tekton-releases/github.com/tektoncd/pipeline/cmd/controller   <none>                                                                       2932d83e8134   9 days ago      84.1MB
gcr.io/tekton-releases/github.com/tektoncd/pipeline/cmd/resolvers    <none>                                                                       9473b5b8afef   9 days ago      87.2MB
lines_appjs                                                          latest                                                                       790e7eb77626   2 weeks ago     660MB
ngrinder/controller                                                  latest                                                                       fc413d0f027b   5 weeks ago     453MB
ngrinder/agent                                                       latest                                                                       6fe65a3190ff   8 weeks ago     173MB
hubproxy.docker.internal:5000/docker/desktop-kubernetes              kubernetes-v1.25.2-cni-v1.1.1-critools-v1.24.2-cri-dockerd-v0.2.5-1-debian   181339469f74   2 months ago    349MB
k8s.gcr.io/kube-apiserver                                            v1.25.2                                                                      0b1fb9b45fa3   2 months ago    123MB
k8s.gcr.io/kube-proxy                                                v1.25.2                                                                      68348500321c   2 months ago    58MB

$ docker push 
```

Docker를 쓸 때의 무엇보다 좋은 점은 애플리케이션이 언제 어디서나 동일한 환경을 유지한다는 것이다. 사용자의 머신에서 정상적으로 실행되면 어느 리눅스 머신에서도
잘 실행된다. 호스트 머신에 Node.js가 설치되어 있는지를 걱정할 필요가 없다.

- 어느 환경에서도 아래와 같이 실행이 가능하다.

```shell
$ docker run -p 8080:8080 -d {Repository Name}
```

# GKE 환경에서 Kubernetes Engine을 활용하기

### GCP 활성화 및 GCloud SDK를 설치한다.

- GCP 활성화는 Billing을 등록하면 되고, 신규 가입자는 무료 사용 기간에 따른 Credit을 제공한다.

### kubectl 명령행 도구 설치 한다.

```shell 

gcloud components install kubectl

```

### 클러스터 생성 및 노드 조회, 확인하기

```shell 

# 아래의 명령어로 gcloud sdk에서 사용이 가능하다. 
$ gcloud beta container --project "프로젝트명" clusters create "클러스터명" --zone "us-central1-c" --no-enable-basic-auth --cluster-version "1.23.12-gke.100" --release-channel "regular" --machine-type "e2-medium" --image-type "COS_CONTAINERD" --disk-type "pd-balanced" --disk-size "100" --metadata disable-legacy-endpoints=true --scopes "https://www.googleapis.com/auth/devstorage.read_only","https://www.googleapis.com/auth/logging.write","https://www.googleapis.com/auth/monitoring","https://www.googleapis.com/auth/servicecontrol","https://www.googleapis.com/auth/service.management.readonly","https://www.googleapis.com/auth/trace.append" --max-pods-per-node "110" --num-nodes "3" --logging=SYSTEM,WORKLOAD --monitoring=SYSTEM --enable-ip-alias --network "projects/lines-infra/global/networks/default" --subnetwork "projects/lines-infra/regions/us-central1/subnetworks/default" --no-enable-intra-node-visibility --default-max-pods-per-node "110" --no-enable-master-authorized-networks --addons HorizontalPodAutoscaling,HttpLoadBalancing,GcePersistentDiskCsiDriver --enable-autoupgrade --enable-autorepair --max-surge-upgrade 1 --max-unavailable-upgrade 0 --enable-shielded-nodes --node-locations "us-central1-c"

$ kubectl get nodes 

NAME                                           STATUS   ROLES    AGE   VERSION
gke-lines-cluster-default-pool-0f0b3237-bxgp   Ready    <none>   18h   v1.23.12-gke.100
gke-lines-cluster-default-pool-0f0b3237-d5ks   Ready    <none>   18h   v1.23.12-gke.100
gke-lines-cluster-default-pool-0f0b3237-wmnh   Ready    <none>   18h   v1.23.12-gke.100

$ gcloud compute ssh <node-name> # 노드로 로그인해 노드에 무엇이 실행 중인지 살펴볼 수 있다. 
# 해당 Node로 직접 접근이 가능해진다. 처음 접급시에는 접근 계정 비밀번호를 요청한다. 요청 받은 정보에 따라 접근할 수 있다. 

```

### 오브젝트 세부정보 가져오기

```shell 

$ kubectl describe node gke-lines-cluster-default-pool-0f0b3237-wmnh
# 아래의 정보가 출력됨! 

Name:               gke-lines-cluster-default-pool-0f0b3237-wmnh
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/instance-type=e2-medium
                    beta.kubernetes.io/os=linux
                    cloud.google.com/gke-boot-disk=pd-standard
                    cloud.google.com/gke-container-runtime=containerd
                    cloud.google.com/gke-cpu-scaling-level=2
                    cloud.google.com/gke-max-pods-per-node=110
                    cloud.google.com/gke-nodepool=default-pool
                    cloud.google.com/gke-os-distribution=cos
                    cloud.google.com/machine-family=e2
                    cloud.google.com/private-node=false
                    failure-domain.beta.kubernetes.io/region=us-central1
                    failure-domain.beta.kubernetes.io/zone=us-central1-c
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=gke-lines-cluster-default-pool-0f0b3237-wmnh
                    kubernetes.io/os=linux
                    node.kubernetes.io/instance-type=e2-medium
                    topology.gke.io/zone=us-central1-c
                    topology.kubernetes.io/region=us-central1
                    topology.kubernetes.io/zone=us-central1-c
Annotations:        container.googleapis.com/instance_id: 4759028284894128442
                    csi.volume.kubernetes.io/nodeid:
                      {"pd.csi.storage.gke.io":"projects/lines-infra/zones/us-central1-c/instances/gke-lines-cluster-default-pool-0f0b3237-wmnh"}
                    node.alpha.kubernetes.io/ttl: 0
                    node.gke.io/last-applied-node-labels:
                      cloud.google.com/gke-boot-disk=pd-standard,cloud.google.com/gke-container-runtime=containerd,cloud.google.com/gke-cpu-scaling-level=2,clou...
                    node.gke.io/last-applied-node-taints:
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Sat, 03 Dec 2022 18:37:14 +0900
Taints:             <none>
Unschedulable:      false
Lease:
  HolderIdentity:  gke-lines-cluster-default-pool-0f0b3237-wmnh
  AcquireTime:     <unset>
  RenewTime:       Sun, 04 Dec 2022 13:08:13 +0900
Conditions:
  Type                          Status  LastHeartbeatTime                 LastTransitionTime                Reason                          Message
  ----                          ------  -----------------                 ------------------                ------                          -------
  FrequentKubeletRestart        False   Sun, 04 Dec 2022 13:04:20 +0900   Sat, 03 Dec 2022 18:37:16 +0900   NoFrequentKubeletRestart        kubelet is functioning properly
  FrequentDockerRestart         False   Sun, 04 Dec 2022 13:04:20 +0900   Sat, 03 Dec 2022 18:37:16 +0900   NoFrequentDockerRestart         docker is functioning properly
  FrequentContainerdRestart     False   Sun, 04 Dec 2022 13:04:20 +0900   Sat, 03 Dec 2022 18:37:16 +0900   NoFrequentContainerdRestart     containerd is functioning properly
  KernelDeadlock                False   Sun, 04 Dec 2022 13:04:20 +0900   Sat, 03 Dec 2022 18:37:16 +0900   KernelHasNoDeadlock             kernel has no deadlock
  ReadonlyFilesystem            False   Sun, 04 Dec 2022 13:04:20 +0900   Sat, 03 Dec 2022 18:37:16 +0900   FilesystemIsNotReadOnly         Filesystem is not read-only
  CorruptDockerOverlay2         False   Sun, 04 Dec 2022 13:04:20 +0900   Sat, 03 Dec 2022 18:37:16 +0900   NoCorruptDockerOverlay2         docker overlay2 is functioning properly
  FrequentUnregisterNetDevice   False   Sun, 04 Dec 2022 13:04:20 +0900   Sat, 03 Dec 2022 18:37:16 +0900   NoFrequentUnregisterNetDevice   node is functioning properly
  NetworkUnavailable            False   Sun, 04 Dec 2022 02:36:43 +0900   Sun, 04 Dec 2022 02:36:43 +0900   RouteCreated                    NodeController create implicit route
  MemoryPressure                False   Sun, 04 Dec 2022 13:05:57 +0900   Sat, 03 Dec 2022 18:33:54 +0900   KubeletHasSufficientMemory      kubelet has sufficient memory available
  DiskPressure                  False   Sun, 04 Dec 2022 13:05:57 +0900   Sat, 03 Dec 2022 18:33:54 +0900   KubeletHasNoDiskPressure        kubelet has no disk pressure
  PIDPressure                   False   Sun, 04 Dec 2022 13:05:57 +0900   Sat, 03 Dec 2022 18:33:54 +0900   KubeletHasSufficientPID         kubelet has sufficient PID available
  Ready                         True    Sun, 04 Dec 2022 13:05:57 +0900   Sat, 03 Dec 2022 18:37:24 +0900   KubeletReady                    kubelet is posting ready status. AppArmor enabled
Addresses:
  InternalIP:   10.128.0.12
  ExternalIP:   104.197.253.1
  InternalDNS:  gke-lines-cluster-default-pool-0f0b3237-wmnh.us-central1-c.c.lines-infra.internal
  Hostname:     gke-lines-cluster-default-pool-0f0b3237-wmnh.us-central1-c.c.lines-infra.internal
Capacity:
  attachable-volumes-gce-pd:  15
  cpu:                        2
  ephemeral-storage:          98831908Ki
  hugepages-1Gi:              0
  hugepages-2Mi:              0
  memory:                     4025932Ki
  pods:                       110
Allocatable:
  attachable-volumes-gce-pd:  15
  cpu:                        940m
  ephemeral-storage:          47060071478
  hugepages-1Gi:              0
  hugepages-2Mi:              0
  memory:                     2880076Ki
  pods:                       110
System Info:
  Machine ID:                 98ba85889e472221ab6ebb48fac36520
  System UUID:                98ba8588-9e47-2221-ab6e-bb48fac36520
  Boot ID:                    a9bf8280-7357-4192-9a51-90a7132b2541
  Kernel Version:             5.10.133+
  OS Image:                   Container-Optimized OS from Google
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  containerd://1.5.13
  Kubelet Version:            v1.23.12-gke.100
  Kube-Proxy Version:         v1.23.12-gke.100
PodCIDR:                      10.120.0.0/24
PodCIDRs:                     10.120.0.0/24
ProviderID:                   gce://lines-infra/us-central1-c/gke-lines-cluster-default-pool-0f0b3237-wmnh
Non-terminated Pods:          (5 in total)
  Namespace                   Name                                                       CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
  ---------                   ----                                                       ------------  ----------  ---------------  -------------  ---
  kube-system                 fluentbit-gke-xr8f4                                        100m (10%)    0 (0%)      200Mi (7%)       500Mi (17%)    18h
  kube-system                 gke-metrics-agent-2nm8v                                    8m (0%)       0 (0%)      100Mi (3%)       100Mi (3%)     18h
  kube-system                 kube-proxy-gke-lines-cluster-default-pool-0f0b3237-wmnh    100m (10%)    0 (0%)      0 (0%)           0 (0%)         18h
  kube-system                 metrics-server-v0.5.2-866bc7fbf8-kf2tf                     48m (5%)      43m (4%)    105Mi (3%)       355Mi (12%)    18h
  kube-system                 pdcsi-node-4bd8g                                           10m (1%)      0 (0%)      20Mi (0%)        100Mi (3%)     18h
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource                   Requests     Limits
  --------                   --------     ------
  cpu                        266m (28%)   43m (4%)
  memory                     425Mi (15%)  1055Mi (37%)
  ephemeral-storage          0 (0%)       0 (0%)
  hugepages-1Gi              0 (0%)       0 (0%)
  hugepages-2Mi              0 (0%)       0 (0%)
  attachable-volumes-gce-pd  0            0
Events:
  Type     Reason            Age                From            Message
  ----     ------            ----               ----            -------
  Warning  NodeSysctlChange  31m (x4 over 18h)  sysctl-monitor  {"unmanaged": {"net.netfilter.nf_conntrack_buckets": "32768"}}

```

### Kubectl Alias 설정

```shell 

$ vi ~/.zshrc # 본인이 사용하는 Shell에 맞춰 등록 

# K8S Setting
alias k=kubectl

```

### Kubectl tab 자동완성

약간 이런 느낌으로 세팅해야함

```shell 

$ k describe
debug     -- Create debugging sessions for troubleshooting workloads and nodes
delete    -- Delete resources by file names, stdin, resources and names, or by resources and label selector
describe  -- Show details of a specific resource or group of resources

```

- 자동완성으로 shell을 세팅하려면 아래와 같이 zsh | bash 에 설정 해야함.

```shell

# K8S Setting
alias k=kubectl
source <(kubectl completion zsh) ## 이게 추가 되어야 함! 

```

- zshrc
    - https://kubernetes.io/docs/tasks/tools/included/optional-kubectl-configs-zsh/
- bashrc
    - https://kubernetes.io/docs/tasks/tools/included/optional-kubectl-configs-bash-linux/

### Image 를 생성 후, Google Container Registry 에 업로드 하기

```shell

$ docker build . -t lines_admin_nextjs -f /Users/lines/sources/01_bonggit/admin-site/frontend/deployments/Dockerfile

```


```shell

$ docker run --rm -it -p 8080:3000 --name lines-admin-front lines_admin_nextjs:latest

```

```shell

$ docker tag lines_admin_nextjs gcr.io/lines-infra/lines_admin_front:v1.0

```

```shell

$ docker push gcr.io/lines-infra/lines_admin_front:v1.0
The push refers to repository [gcr.io/lines-infra/lines_admin_front]
4ac7e340b9ea: Pushed
554c25c70525: Pushed
17ce195f1be2: Pushed
e98f1e5696ab: Pushed
c0eb9dbb3b12: Pushed
a34c94d043a9: Pushed
c7540bf7dad1: Pushed
57712d92e078: Layer already exists
e3b722a14653: Layer already exists
75b8cff8c6ab: Layer already exists
17bec77d7fdc: Layer already exists
v1.0: digest: sha256:1f79a61939bdaf706b660e4d0be147eac3b3483c3c12c54ed66e99858903b435 size: 2627

```

```shell
$ kubectl run lines-cluster --image=gcr.io/lines-infra/lines_admin_front:v1.0 --port=3000                   
pod/lines-cluster created
```

```shell
$ kubectl get pods
```

```shell

# 아래의 명령어는 동작하지 않음. 
$ kubectl expose rc lines-cluster --type=LoadBalancer --name lines-admin-front-http

# 변경 
$ kubectl expose pod lines-cluster --type=LoadBalancer --name lines-admin-front-http

```

### 진행 과정 상의 에러 해결 Tips

- kubectl expose rc lines-cluster --type=LoadBalancer --name lines-admin-front-http 동작하지 않는 경우

```shell

$ kubectl expose rc lines-cluster --type=LoadBalancer --name lines-admin-front-http
Error from server (NotFound): replicationcontrollers "lines-cluster" not found

```

> [Loadbalancer 연동 실습](https://velog.io/@roon-replica/k8s-2-b.-k8s-%EC%8B%A4%EC%8A%B5)

- kubectl run lines-cluster --image=gcr.io/lines-infra/lines_admin_front:v1.0 --port=8080 동작하지 않는 경우

> [Stack Over Flow 해결 가이드](https://stackoverflow.com/questions/72042794/when-creating-pod-it-go-into-crashloopbackoff-logs-show-exec-usr-local-bin-do)

```shell

exec /usr/local/bin/docker-entrypoint.sh: exec format error 

```

- gcr.io/lines-infra/lines_admin_front:v1.0 동작하지 않는 경우

> [권한 세팅 가이드 GCP](https://cloud.google.com/container-registry/docs/advanced-authentication)

```shell 

$ gcloud auth activate-service-account 948099043330-compute@developer.gserviceaccount.com  --key-file=./lines-infra-c67d9a1eafc9.json
Activated service account credentials for: [948099043330-compute@developer.gserviceaccount.com]

$ gcloud auth configure-docker
Adding credentials for all GCR repositories.
WARNING: A long list of credential helpers may cause delays running 'docker build'. We recommend passing the registry name to configure only the registry you are using.
After update, the following will be written to your Docker config file located at [/Users/lines/.docker/config.json]:
 {
  "credHelpers": {
    "gcr.io": "gcloud",
    "us.gcr.io": "gcloud",
    "eu.gcr.io": "gcloud",
    "asia.gcr.io": "gcloud",
    "staging-k8s.gcr.io": "gcloud",
    "marketplace.gcr.io": "gcloud"
  }
}

Do you want to continue (Y/n)?  Y

Docker configuration file updated.

```

- GKE의 Cluster로 접근이 안되는 경우,

```shell 

$ kubectl get nodes
The connection to the server kubernetes.docker.internal:6443 was refused - did you specify the right host or port?

```

[해결방안](https://cloud.google.com/kubernetes-engine/docs/troubleshooting)

하지만 아래와 같이 하면 바로 동작하지는 않음...

```shell

sudo gcloud container clusters get-credentials lines-cluster
ERROR: (gcloud.container.clusters.get-credentials) One of [--zone, --region] must be supplied: Please specify location.

```

실제 GKE 클러스터가 구성된 Region을 같이 붙여서 사용한다.

> 참고로 Region을 기본으로 설정해두는 Config가 있을 것 같지만 우선은 아래와 같이 했다.

```shell 

$ sudo gcloud container clusters get-credentials lines-cluster --region=us-central1-c
Fetching cluster endpoint and auth data.
kubeconfig entry generated for lines-cluster.

```

kubecetl 명령어 동작확인

```shell 

kubectl get nodes
W1204 12:56:24.439537   22454 gcp.go:119] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.26+; use gcloud instead.
To learn more, consult https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke
NAME                                           STATUS   ROLES    AGE   VERSION
gke-lines-cluster-default-pool-0f0b3237-bxgp   Ready    <none>   18h   v1.23.12-gke.100
gke-lines-cluster-default-pool-0f0b3237-d5ks   Ready    <none>   18h   v1.23.12-gke.100
gke-lines-cluster-default-pool-0f0b3237-wmnh   Ready    <none>   18h   v1.23.12-gke.100

```

> [Kubernetes Docs](https://kubernetes.io/docs/)