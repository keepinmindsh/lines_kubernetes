
# Pod

## Lifecycle

### Pod Lifecycle

Pod 의 기본적인 라이프사이클은 Pending Phase 에서 시작해서, 만약 주요한 컨테이너의 최소 한 개의 상태가 OK로 시작되면  Running Phase 로 변경된다.  
그런 다음 파드의 컨테이너가 실패로 종료되었는지 여부에 따라 Succeeded 또는 Failed 로 변경된다.

- **Pending**
    - 파드가 쿠버네티스 클러스터에서 승인되었지만, 하나 이상의 컨테이너가 설정되지 않았고 실행할 준비가 되지 않았다. 여기에는 파드가 스케줄되기 이전까지의 시간 뿐만 아니라 네트워크를 통한 컨테이너 이미지 다운로드 시간도 포함된다.
        - ReadinessProbe
        - Policy
- **Running**
    - 파드가 노드에 바인딩되었고, 모든 컨테이너가 생성되었다. 적어도 하나의 컨테이너가 아직 실행 중이거나, 시작 또는 재시작 중에 있다.
        - LivenessProbe
        - Qos
- **Succeeded**
    - 파드에 있는 모든 컨테이너들이 성공적으로 종료되었고, 재시작되지 않을 것이다.
        - Policy
- **Failed**
    - 파드에 있는 모든 컨테이너가 종료되었고, 적어도 하나 이상의 컨테이너가 실패로 종료되었다. 즉, 해당 컨테이너는 non-zero 상태로 빠져나왔거나(exited) 시스템에 의해서 종료(terminated)되었다.
        - Policy
- **Unknown**
    - 어떤 이유에 의해서 파드의 상태를 얻을 수 없다. 이 단계는 일반적으로 파드가 실행되어야 하는 노드와의 통신 오류로 인해 발생한다.

### Container Status

- **Waiting**
    - 만약 컨테이너가 Running 또는 Terminated 상태가 아니면, Waiting 상태이다. Waiting 상태의 컨테이너는 시작을 완료하는 데 필요한 작업(예를 들어, 컨테이너 이미지 레지스트리에서 컨테이너 이미지 가져오거나, 시크릿(Secret) 데이터를 적용하는 작업)을 계속 실행하고 있는 중이다. kubectl 을 사용하여 컨테이너가 Waiting 인 파드를 쿼리하면, 컨테이너가 해당 상태에 있는 이유를 요약하는 Reason 필드도 표시된다.

- **Running**
    - Running 상태는 컨테이너가 문제없이 실행되고 있음을 나타낸다. postStart 훅이 구성되어 있었다면, 이미 실행되고 완료되었다. kubectl 을 사용하여 컨테이너가 Running 인 파드를 쿼리하면, 컨테이너가 Running 상태에 진입한 시기에 대한 정보도 볼 수 있다.

- **Terminated**
    - Terminated 상태의 컨테이너는 실행을 시작한 다음 완료될 때까지 실행되었거나 어떤 이유로 실패했다. kubectl 을 사용하여 컨테이너가 Terminated 인 파드를 쿼리하면, 이유와 종료 코드 그리고 해당 컨테이너의 실행 기간에 대한 시작과 종료 시간이 표시된다.

### ContainerRestartPolicy

restartPolicy 는 파드의 모든 컨테이너에 적용된다.   
restartPolicy 는 동일한 노드에서 kubelet에 의한 컨테이너 재시작만을 의미한다.   
파드의 컨테이너가 종료된 후, kubelet은 5분으로 제한되는 지수 백오프 지연(10초, 20초, 40초, …)으로 컨테이너를 재시작한다.    
컨테이너가 10분 동안 아무런 문제없이 실행되면, kubelet은 해당 컨테이너의 재시작 백오프 타이머를 재설정한다.

- **restartPolicy**
    - Always, OnFailure, Never
    - 기본 값 : Always

### Pod Condition

- **PodScheduled**
    - 파드가 노드에 스케줄되었다.
- **ContainersReady**
    - 파드의 모든 컨테이너가 준비되었다.
- **Initialized**
    - 모든 초기화 컨테이너가 성공적으로 완료(completed)되었다.
- **Ready**
    - 파드는 요청을 처리할 수 있으며 일치하는 모든 서비스의 로드 밸런싱 풀에 추가되어야 한다.

### References

```yaml
status:
  phase: Pending  # Pending, Running, Succeeded, Failed, UnKnown 
  conditions: # Initialized , ContainerReady, PodScheduled, Ready 
  - type: Initialized
    status: 'True'
    lastProbeTime: null 
    lastTransitionTime: '2019-09-26T22:07:56Z'
  - type: PodScheduled 
    status: 'True'
    lastProbeTime: null 
    lastTransitionTime: '2019-09-26T22:07:56Z'
  - type: ContainersReady
    status: 'False'
    lastProbeTime: null
    lastTransitionTime: '2019-09-26T22:08:11Z'
    reason: ContainersNotReady # ContainerNotReady, PodCompleted 
  - type: Ready
    status: 'False'
    lastProbeTime: null
    lastTransitionTime: '2019-09-26T22:08:11Z'
    reason: ContainersNotReady
  containerStatuses:
  - name: container 
    state: # Waiting, Running, Terminated  
      waiting: 
        reason: ContainerCreating  # ContainerCreating, CrashLoopBackOff, Error, Completed 
    lastState: {}
    ready: false
    restartCount: 0 
    image: tmkube/init
    imageID: ''
    started: false
```

- Pod 의 최초상태는 Pending
    - Container 가 기동되기 전에 설정되어야할 항목이 있음
    - Pending 상태에서 Failed 로 상태가 변경될 수 있음
    - Pending 상태에서 Unknown 으로 상태가 변경될 수 있음

```yaml
apiVersion: v1 
kind: Pod 
metadata: 
  name: myapp-pod 
  labels: 
    app: myapp
spec: 
  containers: 
  - name: myapp-container
    image: busybox:1.28 
    command: [ 'sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers: # 본 Container 보다 먼저 실행되었다고 판단하는 것 
  - name: init-myservice 
    image: busybox:1.28 
    command: [ 'sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb 
    image: busybox:1.28
    command: [ 'sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```

- Pod 의 그 다음 단계는 Running
    - Running 단게에서는 Unknown으로 변경될 수 있음
    - Unkown 상태는 Failed 상태로 변경이 가능함.

- Pod 의 마지막 상태는 Failed 또는 Succeeded

- 마지막으로 Pod의 상태는 무조건
    - ContainerReady : False
    - Ready : False

### Pod ( ReadinessProbe, LivenessProbe )

기본적으로 Pod를 종료할 때 별다른 설정 없이 종료하게 되면, 종료되기 전까지의 시점에 일부 요청건에 대해서 장애가 발생할 수 있는데,   
그 이유는 기존의 Pod가 종료되기전, 신규 Pod 가 구성된 시점에는 아직 정상적동하지 않는 Pod일 수 있기 때문에 (내부의 소스 기동 시간 등등 ),  
50:50의 확률로 서비스가 정상적으로 종료가 안된 Pod로 요청이 접수 될 수 있는데, 이건 신규 Pod가 기동되는 시점도 동일하게 발생할 수 있는 문제이다.

- ReadinessProbe
    - App 구동 순간에 트래픽 실패를 없앰

- LivenessProbe
    - App 장애시 지속적인 트래픽 실패를 없앰

![Deployments](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/ProbesSettings01.png)

![Deployments](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/ProbesSettings02.png)


## QoS classes

설정에 따라 파드에 필요한 성능을 관리하는 방식을 제공함.  
예) 3개의 파드 중 1개의 파드에 성능이 더 필요할 경우 3개중 하나를 죽이고 파드 하나로 성능을 올려주는 역할을 설정할 수 있음.

![Deployments](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/qos_class.png)

```yaml
kind: Prod 
spec:
  containers:
    - resources:
        requests:
          memory: 1G
          cpu: 2
        limits:
          memory: 2G
          cpu: 4
```

- Guaranteed
- Burstable
    - OOM Score
- BestEffort

> [Assign Memory Resources to Containers and Pods](https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource/)

## Node Scheduling

- Node 선택 : 원하는 노드로 자원 할당이 되도록 관리할 수 있음
    - NodeName
    - NodeSelector
    - NodeAffinity
        - matchExpressions
            - required, preferred
                - key
                - operator : Exists, DoesNotExist, In, NotIn, Gt, Lt

- Pod 간 집중/분산
    - Pod Affinity
        - podAffinity
            - matchExpressions
                - key : type
                - operator : ~
                - values:[~]
        - topologyKey
    - Anti-Affinity
        - podAntiAffinity

- Node에 할당 제한
    - Toleration / Taint
        - Pod
            - Toleration
                - key
                - operator : Equal, Exists
                - value
                - effect : NoSchedule, PreferNoSchedule, NoExecute
    - Pod가 Toleration을 설정해야 Taint로 설정된 Node로 Pod 세팅이 가능함. 