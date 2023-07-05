# Section 17 - 파드의 컴퓨팅 리소스 관리 

## Cheat sheet 

```shell
# Create a new resource quota named my-quota
kubectl create quota my-quota --hard=cpu=1,memory=1G,pods=2,services=3,replicationcontrollers=2,resourcequotas=1,secrets=5,persistentvolumeclaims=10

# Create a new resource quota named best-effort
kubectl create quota best-effort --hard=pods=100 --scopes=BestEffort
```

## 파드 컨테이너의 리소스 요청

파드를 생성할 때 컨테이너가 필요로 하는 CPU와 메모리 양과 사용할 수 있는 엄격한 제한을 지정할 수 있다.  
이들은 컨테이너에 개별적으로 지정되며 파드 전체에 지정되지 않는다.   
파드의 리소스 요청과 제한은 모든 컨테이너의 리소스 요청과 제한의 합이다.  

### 리소스 요청을 갖는 파드 생성하기 

```yaml
apiVersion: v1 
kind: Pod 
metadata: 
  name: requests-pod 
spec: 
  containers: 
  - image: busybox 
    command: ["dd", "if=/dev/zero", "of=/dev/null"]
    name: main 
    resources: 
      requests: 
        cpu: 200m 
        memory: 10Mi 
```

파드 매니페스트에서 컨테이너를 제대로 실행하려면 1/5 CPU 코어를 필요로 한다.  
이런 파드/컨테이너 다섯 개를 CPU 코어 하나에서 충분히 빠르게 실행할 수 있다.  
CPU 요청을 지정하지 않으면 컨테이너에서 실행 중인 프로세스에 할당되는 CPU 시간에 신경 쓰지 않는다는 것과 같다.  
최악의 경우 CPU 시간을 전혀 할당받지 못할 수 있다. 시간이 중요하지 않은 우선순위가 낮은 배치작업은 괜찮지만  
사용자 요청을 처리하는 컨테이너에는 분명 적합하지 않다. 

```shell
$ kubectl exec -it requests-pod top 
```

컨테이너에서 실행한 dd 명령은 사용할 수 있는 최대 CPU를 소비하지만 스레드 하나로 실행되므로 코너 하나만을 사용할 수 있다.  
이 예제가 실행하는 Minikube 가상머신은 CPU 코어 두 개가 할당돼 있다. 이것은 프로세스가 전체 CPU의 50%를 소비하는 것으로  
표시하는 이유다. 

## 컨테이너에 사용 가능한 리소스 제한 

### 컨테이너가 사용 가능한 리소스 양을 엄격한 제한으로 설정 

다른 모든 프로세스가 유휴 상태일 때 컨테이너가 남은 모든 CPU를 사용하는 방법을 알아봤다. 그러나 특정 컨테이너가 지정한 CPU 양보다 많은 CPU 를 사용하는 것을 막고  
싶을 수 있다. 그리고 컨테이너가 사용하는 메모리 양을 제한하고 싶을 수도 있다. CPU의 경우 압축 가능한 리소스로서 처리중인 프로세스에 부정적인 영향을 주지 않고 
컨테이너가 사용하는 CPU 양을 조정할 수 있다. 메모리는 분명 다른다. 압축이 불가능하다. 프로세스에 메모리가 주어지면 프로세스가 메모리를 해제하지 않는 한 
가져갈 수 없다. 그것이 컨에티너에 할당되는 메모리의 최대량을 제한해야 하는 이유다.  

- 리소스 제한을 갖는 파드 생성 

```yaml
apiVersion: v1 
kind: Pod 
metadata: 
  name: limited-pod 
spec:
  containers:
  - image: busybox 
    command: ["dd", "if=/dev/zero", "of=/dev/null"]
    name: main 
    resources: 
      limits: 
        cpu: 1
        memory: 20Mi 
```

위의 파드의 컨테이너는 CPU와 메모리에 리소스 제한이 설정돼 있다. 컨테이너 내부에 실행 중인 프로세스는 CPU 1코어와 메모리 20Mi 이상을 사용할 수 없다. 

- 리소스 제한 오버커밋 

리소스 요청과 달리 리소스 제한은 노드의 할당 가능한 리소스 양으로 제한 되지 않는다. 노드에 ㅇㅆ는 모든 파드의 리소스 제한 합계는 노드 용량의 100%를 초과할 수 있다. 
다시 말하면 리소스 제한은 오버커밋될 수 있다. 이는 중요한 결과를 갖는다. 노드 리소스가 100%가 다 소진되면 특정 컨테이너는 제거돼야 한다. 

### 리소스 제한 초과 

메모리는 CPU와 달리 프로세스가 제한보다 많은 메모리를 할당 받으려 시도하면 프로세스는 종료된다. 컨테이너가 OOMKilled 됐다고 한다. OOM은 Out Of Memory다. 
컨테이너가 종료되는 것을 원치 않는다면 메모리 제한을 너무 낮게 설정하지 않는 것이 중요하다. 그러나 컨테이너는 제한을 넘어서지 않아도 OOMKilled 될 수 있다. 

### 컨테이너의 애플리케이션 제한을 바라보는 방법 

- 컨테이너는 항상 컨테이너 메모리가 아닌 노드 메모리를 본다. 

top 명령은 컨테이너가 실행 중인 전체 노드의 메모리 양을 표시한다. 컨테이너에 사용 가능한 메모리의 제한을 설정하더라도 
컨테이너는 이 제한을 인식하지 못한다. 

- 컨테이너는 또한 노드의 모든 CPU 코어를 본다. 

예)  
64 코어 CPU에 실행 중인 1코어 CPU 제한의 컨테이너는 전체 CPU 시간의 1/64을 얻는다. CPU 제한이 1코어로 설정되더라도 컨테이너의 프로세스는 한개 코어에서만 
실행되는 것이 아니다. 


## 파드 QoS 클래스 이해

- 파드의 세가지 서비스 품질(QOS, Quality Of Service)
  - Best Effort ( 최하위 우선 순위 )
    - 우선 순위가 가장 낮은 클래스는 BestEffort 클래스다. 아무런 리소스 요청과 제한이 없는 파드에 할당된다. 
  - Burstable 
    - 모든 리소스를 컨테이너의 리소스 요청이 리소스 제한과 동일한 파드에 주어진다. 
      - 동작 조건 
        - CPU와 메모리에 리소스 요청과 제한이 모두 설정돼야 한다. 
        - 각 컨테이너에 설정돼야 한다. 
        - 리소스 요청과 제한이 동일해야 한다. 
  - Guaranteed ( 최상위 우선 순위 )
    - BestEffort와 Guaranteed 사이가 Burstable QoS 클래스다. 그 밖의 다른 파드는 이 클래스에 해당한다. 

### 컨테이너 QoS 클래스 파악 

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_qos_001.png)

### 다중 컨테이너 파드의 QoS 클래스 파악 

다중 컨테이너의 파드의 경우 모든 컨테이너가 동일한 QoS 클래스를 가지만 그것이 파드의 QoS가 된다. 

### 메모리가 부족할 때 어떤 프로세스가 종료되는지 이해 

- QoS 클래스가 우선순위르리 정하는 방법 이해 

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_qos_002.png)

- BestEffort 파드에 실행 중인 프로세스가 Burstable 파드의 프로세스 이전에 항상 종료된다. 
- 물론 BestEffort 파드의 프로세스는 Guaranteed 파드의 프로세스가 종료되기 전에 종료된다. 
- 이와 마찬가지로 Burstable 파드의 프로세스도 Guaranteed 파드 이전에 종료된다. 

### 동일한 QoS 클래스를 갖는 컨테이너를 다루는 방법 이해 

실행 중인 각 프로세스는 OOM 점수를 갖는다. 시스템은 모든 실행 중인 프로세스의 OOM 점수를 비교해 종료할 프로세스를 선정한다. 메모리 해제가 필요하면 
가장 높은 점수의 프로세스가 종료된다. 

## 네임스페이스별 파드에 대한 기본 요청과 제한 설정


### LimitRange 

모든 컨테이너에 리소스 요청과 제한을 설정ㅎ나는 대신 LimitRange 리소스를 생성해 이를 수행할 수 있다. 

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_qos_003.png)

```yaml
apiVersion: v1 
kind: LimitRange
metadata: 
  name: example 
spec: 
  limits: 
  - type: Pod 
    min: 
      cpu: 50m
      memory: 5Mi 
    max: 
      cpu: 1 
      memory: 1Gi 
  - type: Container 
    defaultRequest: 
      cpu: 100m 
      memory: 10Mi 
    default: 
      cpu: 200m 
      memory: 10Mi 
      min:
        cpu: 1
        memory: 1Gi 
      maxLimitRequestRatio: 
        cpu: 4 
        memory: 10
  - type: PersistentVolumeClaim 
    min: 
      storage: 1Gi 
    max:  
      storage: 10Gi 
```

<각 항목별 설명에 대한 이미지>

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_qos_004.png)

### 강제 리소스 제한 

제한이 설정되면 이제 LimitRange에서 허용하는 것보다 더 많은 CPU를 요청하는 파드를 만들 수 있다. 

```yaml 
resources:
  requests: 
    cpu: 2 
```

파드의 컨테이너가 이전에 설정한 LimitRange의 최대보다 큰 두 개의 CPU를 요청한다. 

```shell
$ kubectl create -f limits-pod-too-big.yaml 
Error from server (Forbidden): error when creating " limits - pod - too - big.yaml" :
pods "too - big " is forbidden : [
maximum cpu usage per Pod is 1, but request is 2 .,
maximum cpu usage per Container is but request is 2. ]
```

- 컨테이너가 CPU 두 개를 요청했지만 컨테이너의 최대 CPU 제한은 하낟, 
- 마찬가지로 파드가 CPU 두 개를 요청했지만 최대 CPU는 하나다. 

### 기본 리소스 요청과 제한 적용 

기본 리소스 요청과 제한이 리소스 요청과 제한을 지정하는 않은 컨테이너에 적용되는 방법을 살펴본다. 

```shell 
$ kubectl create -f ./kubia-manual.yaml 
```

```shell 
$ kubectl describe po kubia-manual 
```

컨테이너의 요청과 제한이 Limit Range 오브젝트에서 지정한 것과 일치한다. 또 다른 네임 스페시으에 다른 Limit Range 스펙을 사용한다면 그 네임 스페이스의 파드는 분면 다른 요청과 제한으로 생성된다. 이는 관리자가 네임스페이스당 파드의 기본, 최소, 최대 리소스를 설정하도록 한다.   

동일 쿠버네티스 클러스터에서 실행하는 개발, QA, 스테이징, 프로덕션 파드를 분리하는 경우 각 네임스페이스에 다른 LimitRange를 사용하면 큰 파드는 특정 네임스페이스에만 생성되는 반면 나머지 네임스페이스는 더 작은 파드로 제한되도록 보장한다.   

그러나 LimitRange에 설정된 제한은 각 개별 파드/컨테이너에만 적용된다는 것을 기억하라. 많은 수의 파드를 만들어 클러스터에서 사용 가능한 모든 리소스를 써버릴 수 있다. 

## 네임스페이스의 사용 가능한 총 리소스 제한하기

LimitRange는 개별 파드에만 적용되지만 클러스터 관리자는 네임스페이스에서 사용 가능한 총 리소스 양을 제한할 수 있는 방법이 필요하다. 

### 리소스쿼터 오브젝트 

- CPU 및 메모리에 관한 리소스 쿼터 생성 

```yaml 
apiVersion: v1 
kind: ResourceQuota 
metadata: 
  name: cpu-and-mem
spec: 
  hard: 
    requests.cpu: 400m 
    requests.memory: 200Mi 
    limits.cpu: 600m 
    limits.memory: 500Mi 
```

이 리소스 쿼터는 네임스페이스에서 파드가 요청할 수 있는 최대 CPU를 400밀리코어로 설정한다. 네임스페이스의 최대 총 CPU 제한은 600 밀리코어로 설정된다. 메모리의 경우 최대 총 요청은 200MiB로 설정되고 제한은 500MiB로 설정된다. 

- 쿼터와 쿼터 사용량 검사 

```shell 
$ kubectl describe quota 
```

- 리소스 쿼터와 함께 LimitRange 생성 

리소스 쿼터를 생성할 때 주의할 점은 LimitRange 오브젝토도 함께 생성해야 한다는 것이다. 

```shell 
$ kubectl create -f kubia-manual.yaml 
```

> 특정 리소스에 대한 쿼터가 설정된 경우, 파드에는 동일한 리소스에 대한 요청 제한이 설정돼야 한다.  
> 그렇지많으면 API 서버가 파드를 허용하지 않는다. 

### 퍼시스턴스 스토리지에 관한 쿼터 지정하기 

```yaml 
apiVersion: v1 
kind: ResourceQuota 
metadata:
  name: storage 
spec:
  hard:
    requests.storage: 500Gi 
    ssd.storageclass.storage.k8s.io/requests.storage: 300Gi 
    standard.storageclass.storage.k8s.io/requests.storage: 1Ti 
```

### 생성 가능한 오브젝트 수 제한 

리소스 쿼터는 네임스패에스내의 파드, 레플리케이션 컨트롤러, 서비스 및 그 외의 오브젝트 수를 제한하도록 구성할수 있다.  
이를 통해 클러스터 관리자는 예를 들어 사용자의 결재 플랜에 따라 생성할 수 있는 오브젝트 수를 제한할 수 있으며, 서비스가 사용할 수 있는 퍼블릭 IP 또는 노드포트 수를 제한할 수 있다.  

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_qos_005.png)

위의 예제의 리소스 쿼터를 사용하면 사용자가 직접 또는 레플리케이션 컨트롤러, 레플리카셋, 데몬세스 잡 등으로 생성되는지 여부에 관계없이 네임스페이스에서 최대 열개의 파드를 생성할 수 있다. 

오브젝트 수의 쿼터는 아래의 오브젝트 설정 가능 

- 파드 
- 레플리케이션 컨트롤러 
- 시크릿 
- 컨피그맵 
- 퍼시스턴스볼륨클레임 
- 서비스, 로드 밸런서 서비스와 노드포트 서비스와 같은 두가지 유형의 서비스 

### 특정 파드나 QoS 클래스에 대한 쿼터 지정 

지금까지 생성한 쿼터는 파드의 현재 상태 및 QoS 클래스에 관계없이 모든 파드에 적용된다. 그러나 쿼터는 쿼터 범위로 제한될 수도 있다.  
현재 BestEffort, NotBestEffort, Terminating, NotTerminating의 네가지 범위를 사용할 수 있다. 

```yaml 
apiVersion: v1 
kind: ResourceQuata 
metadata:
  name: besteffort-notterminating-pods 
spec:
  scopes:
  - BestEffort 
  - NotTerminating 
  hard:
    pods: 4
```

위의 쿼터는 유효 데드라인이 없는 BestEffort QoS 클래스가 갖는 파드가 최대 4개 존재하도록 보장한다. 그 대신 쿼터가 NotBestEffort 파드를 대상으로 하는 경우 request.cpu, reuqest.memory limit.cpu 및 limit.memory도 지정할 수 있다. 

## 파드 리소스 사용량 모니터링 

### 실제 리소스 사용량 수집과 검색 

Kubelet 자체에는 이미 cAdvisor 라는 에이전트가 포함돼 있는데 이 에이전트는 노드에서 실행되는 개별 컨테이너와 노드 전체 리소스 사용데이터를 수집한다. 전체 클러스터를 이러한 통계를 중앙에서 수집하려면 힙스터라는 추가 구성요소를 실행해야 한다.  


![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_qos_006.png)

그림의 화살표는 메트릭 데이터 흐름을 보여준다.  

- 힙스터 활성화 

```shell 
$ minikube addos enable heapster
```

- 클러스터 노드의 CPU 및 메모리 사용량 표시 

```shell
$ kubectl top node 
```

- 개별 파드의 CPU와 메모리 사용량 표시 


```shell 
$ kubectl top pod --all-namesapces
```

### 기간별 리소스 사용량 통계 저장 및 분석 

인플럭스 DB는 애플리케이션 메트릭과 기타 모니터링 데이터를 저장하는데 이상적인 오픈 소스 시계열 데이터 베이스이다. 오픈 소스인 그라파나는 인플럭스 DB에 저장된 데이터를 시작화 하고 시간이 지남에 따라 애플리케이션의 리소스 사용량이 어떻게 변하는지 확인할수 있다. 

- 인플럭스 DB 
- 그라파나 