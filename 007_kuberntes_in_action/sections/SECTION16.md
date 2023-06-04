# Section 16 - 클러스터 노드와 네트워크 보안 

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


## 컨테이너의 보안 컨텍스트 구성
## 파드의 보안 관련 기능 사용 제한
## 파드 네트워크 격리  