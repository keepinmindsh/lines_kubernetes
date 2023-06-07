# Section 17 - 파드의 컴퓨팅 리소스 관리 

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

## 네임스페이스의 사용 가능한 총 리소스 제한하기
## 파드 리소스 사용량 모니터링 