# Section 16 - 클러스터 노드와 네트워크 보안 

## Cheat Sheet 

```shell
# Create a role named "pod-reader" that allows user to perform "get", "watch" and "list" on pods
kubectl create role pod-reader --verb=get --verb=list --verb=watch --resource=pods

# Create a role named "pod-reader" with ResourceName specified
kubectl create role pod-reader --verb=get --resource=pods --resource-name=readablepod --resource-name=anotherpod

# Create a role named "foo" with API Group specified
kubectl create role foo --verb=get,list,watch --resource=rs.extensions

# Create a role named "foo" with SubResource specified
kubectl create role foo --verb=get,list,watch --resource=pods,pods/status
```

## 파드에서 호스트 노드의 네임스페이스 사용

파드의 컨테이너는 일반적으로 별도의 리눅스 네임스페이스에서 실행되므로 프로세스가 다른 컨테이너 또는 노드의 기본 네임스페이스에서 실행 중인 프로세스와 분리된다.  

예를 들어 각 파드는 고유한 네트워크 네임 스페이스를 사용하기 때문에 고유한 IP와 포트공간을 얻는 다는 것을 알고 있을 것이다. 마찬가지로 각 파드는 고유한 PID 네임스페이스가 있기 때문에 
고유한 프로세스 트리가 있으며 고유한 IPC 네임스페이스도 사용하므로 동일한 파드의 프로세스 간 통신 메커니즘 (IPC) 으로 서로 통신할 수 있다. 

### 파드에서 노드의 네트워크 네임스페이스 사용  

특정 파드는 호스트의 기본 네임스페이스에서 작동해야 노드의 리소스와 장치를 읽고 조작할 수 있다. 예를 들어 파드는 가상 네트워크 어댑터 대신 노드의 실제 네트워크 어댑터를 사용해야 할 수도 있다.  
이는 파드 스펙에서 hostNetwork 속성을 true로 설정하면 된다.  

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_network_001.png)

```yaml
apiVersion: v1 
kind: Pod 
metadata: 
    name: pod-with-host-network 
spec: 
    hostNetwork: true 
    containers:
    - name: main 
      image: alpine 
      command: ["/bin/sleep", "999999"]
```

```shell 
$ kubectl exec pod-with-host-network ifconfig 
```

### 호스트 네트워크 네임스페이스를 사용하지 않고 호스트 포트에 바인딩 

파드는 hostNetwork 옵션으로 노드의 기본 네임스페이스의 포트에 바인딩할 수 있지만  
여전히 고유한 네트워크 네임스페이스를 갖는다. 이는 컨테이너의 포트를 정의하는 spec.containers.ports 필드 안에 hostPort 속성을 사용해 할 수 있다.  

hostPort를 사용하는 파드와 NodePort 서비스로 노출된 파드를 혼동하면 안된다.  
파드가 hostPort를 사용하는 경우 노드포트에 대한 연결을 해당 노드에서 실행 중인 파드로 직접 전달되는 반면 NodePort 서비스의 경우 노드포트의 연결은 
임의의 파드로 전달된다. 또 다른 차이점은 hostPort를 사용하는 파드의 경우 노드포트는 해당 파드를 실행하는 노드에만 바인딩되는 반면 NodePort 서비스는  
이런 파드를 실행하지 않는 노드에서도 모든 노드의 포트를 바인딩한다는 것이다.  

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_network_002.png)  

파드가 특정 호스트 포트를 사용하는 경우 두 프로세스가 동일한 호스트 포트에 바인딩될 수 없으므로 파드 인스턴스 하나만 노드에 스케줄링될 수 있다는 점을  
이해해야 한다.  

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_network_003.png)

```yaml
apiVersion: v1 
kind: Pod 
metadata: 
  name: kubia-hostport 
spec: 
  containers: 
  - image: luksa/kubia 
    name: kubia 
    ports: 
    - containerPort: 8080 
      hostPort: 9000 
      protocol: TCP 
```

hostPort 기능은 기본적으로 데몬셋을 사용해 모든 노드에 배포되는 시스템 서비스를 노출하는데 사용된다.

### 노드의 PID와 IPC 네임스페이스 사용 

```yaml
apiVersion: v1 
kind: Pod 
metadata: 
  name: pod-with-host-pid-and-ipc 
spec: 
  hostPID: true 
  hostIPC: true 
  containers: 
  - name: main 
    image: alpine 
    command: ["/bin/sleep", "999999"]
```

파드는 일반적으로 자체 프로세스만 표시하지만 이 파드를 실행한 후 컨테이너의 프로세스 조회하면 
컨테이너에서 실행 중인 프로세스 뿐만 아니라 호스트 노드에서 실행 중인 모든 프로세스가 조회된다. 

```shell
$ kubectl exec pod-with-host-pid-and-ipc ps aux 
```

## 컨테이너의 보안 컨텍스트 구성

파드가 호스트의 리눅스 네임스페이스를 사용하도록 허용하는 것 외에도, 파드 컨텍스트 아래의 개별 컨테이너 스펙에서  
직접 지정할 수 있는 securityContext 속성으로 다른 보안 관련 기능을 파드와 파드의 컨테이너에 구성할 수 있다.  

### 보안 컨텍스트에서 설정할 수 있는 사항 

- 컨테이너의 프로세스를 실행할 사용자 지정하기 
- 컨테이너가 루트로 실행되는 것 방지하기 
- 컨테이너를 특권 모드에서 실행해 노드의 커널에 관한 모든 접근 권한을 가짐 
- 특권 모드에서 컨테이너를 실행해 컨테이너에 가능한 모든 권한을 부여하는 것과 달리 기능을 추가하거나 삭제해 세분화된 권한 구성하기 
- 컨테이너의 권한 확인을 강력하게 하기 위해 SELinux 옵션 설정하기 
- 프로세스가 컨테이너의 파일 시스템에 쓰기 방지하기 

### 컨테이너를 특정 사용자로 실행 

```yaml
apiVersion: v1 
kind: Pod 
metadata: 
  name: pod-as-user-guest 
spec: 
  containers: 
  - name: main 
    image: alpine 
    command: ["/bin/sleep", "999999"]
    securityContext: 
      runAsUse: 405 
```

```shell
$ kubectl get po pod-run-as-non-root 
```

### 특권 모드에서 파드 실행 

노드 커털의 모든 액세스 권한을 얻기 위해 파드의 컨테이너는 특권 모드로 실행된다. 컨테이너의 securityContext 속성에서 privileged 속성을 true로 설정하면 된다.  

```yaml
apiVersion: v1 
kind: Pod 
metadata: 
  name: pod-privileged 
spec:
  containers: 
  - name: main 
    image: alpine 
    command: ["bin/sleep", "999999"]
    securityContext: 
      privileged: true
```

```shell
$ kubectl exec -it pod-with-defaults ls /dev
```

```shell
$ kubectl exec -it pod-privileged ls /dev 
```

### 컨테이너에 개별 커널 기능 추가 

권한 있는 컨테이너를 만들고 무제한 권한을 부여하는 대신 보안 관점에서 훨씬 안전한 방법은 실제로 필요한 커널 기능만 액세스하도록 하는 것이다.  
쿠버네티스를 사용하면 각 컨테이너에 커널 기능을 추가하거나 일부를 삭제할 수 있으므로 컨테이너 권한을 미세조정하고 공격자의 잠재적인 침입의 영향을 제한할 수 있다.  
예를 들어 컨테이너는 일반적으로 시스템 시간을 변경할 수 없다.  

```shell
$ kubectl exec -it pod-with-defaults -- date +%T -s "12:00:00"
```

```yaml
apiVersion: v1 
kind: Pod 
metadata: 
  name: pod-add-settime-capability 
spec: 
  containers: 
  - name: main 
    image: alpine 
    command: ["/bin/sleep", "999999"]
    securityContext: 
      capabilities: 
        add: 
        - SYS_TIME 
```

```shell
$ kubectl exec -it pod-add-settime-capability -- date +%T -s "12:00:00"

$ kubectl exec -it pod-add-settime-capability -- date

$ minikube ssh date 
```

이와 같은 기능을 추가하는 것이 컨테이너에 모든 권한을 부여하는 것보다 훨씬 좋은 방법이다. 

### 컨테이너에서 기능 제거 
### 프로세스가 컨테이너의 파일시스템에 쓰는 것 방지 
### 컨테이너가 다른 사용자로 실행될 때 볼륨 공유 


## 파드의 보안 관련 기능 사용 제한

### PodSecurityPolicy 리소스 소개 

### runAsUser, fsGroup, supplementalGroups 정책 

### allowed, default, disallowed 기능 구성 

### 파드가 사용할 수 잇는 볼륨 우형 제한 

### 각각의 사용자와 그룹에 다른 PodSecurityPolicies 할당 

## 파드 네트워크 격리  

### 네임스페이스에서 네트워크 격리 사용 

### 네임스페이스의 일부 클라이언트 파드만 서버 파드에 연결하도록 허용 

### 쿠버네티스 네임스페이스 간 네트워크 격리 

```yaml 
apiVersion: networking.k8s.io/v1 
kind: NetworkPolicy 
metadata:
  name: shoppingcard-netpolicy 
spec:
  podSelector: 
    matchLables: 
      app: shopping-card 
  ingress: 
  - from: 
    - namespaceSelector: 
        matchLabels: 
          tenant: manning 
      ports:
      - port: 80

```

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_network_004.png)


### CIDR 표기법으로 격리 

NetworkPolicy에서 대상으로 지정된 파드에 액세스할 수 잇는 대상을 정의하려면 파드 또는 네임스페이스 셀렉터를 
지정하는 대신 CIDR 표기법으로 IP 블록을 지정할 수도 있다. 

```yaml 
ingress:
- from: 
  - ipBlock:
    cidr: 192.168.1.0/24 
```

### 파드의 아웃바운드 트래픽 제한 

```yaml 
spec:
  podSelector: 
    macthLabels:
      app: webserver 
    egress: 
    - to: 
      - podSelector: 
          machLabels:
            app: database 
```

- 이 정책은 app=webserver 레이블이 있는 파드에 적용된다. 
- egress : 파드의 아웃바운드 트래픽을 제한한다. 
  - 웹 서버 파드는 app=database 레이블이 있는 파드만 연결할 수 있다. 