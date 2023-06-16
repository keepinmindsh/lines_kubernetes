# Section 19

## 테인트와 톨러레이션을 사용해 특정 노드에서 파드 실행 제한 

고급 스케줄링과 관련된 기능 중 가장 먼저 살펴볼 두가지는 노드 테인트와 이 테인트에 대한 파드 톨러레이션이다.  
테인트와 톨러레이션은 어떤 파드가 특정 노드를 사용할 수 있는지를 제한하고자 사용된다. 

### 테인트와 톨러레이션 소개 

#### 노드와 테인트 표시하기 

마스터 노드에는 테인트가 하나 있다. 테인트에는 키, 값, 효과가 있고 <key>=<value>:<effect> 형태로 표시된다.  
이전 예제에 표시된 마스터 노드의 테인트는 키가 node-role.kubernetes.io/master, 값은 null, 효과는 NoSchedule을 갖는다. 

```shell
$ kubectl describe node master.k8s 
```

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_taint_001.png)

#### 파드의 톨러레이션 표시하기 

kubeadm 으로 설치한 클러스터에는 마스터 노드를 포함해 모든 노드에서 kube-proxy 클러스터 컴포넌트가 파드로 실행된다. 
파드로 실행되는 마스터 노드 컴포넌트도 쿠버네티스 서비스에 접근해야 할 수도 있기 때문에 kube-proxy가 마스터 노드에서도 실행된다. 

```shell
$ kubectl describe po kube-proxy-80wqm -n kube-system 
```

> 파드의 톨러레이션에는 표시되지만 노드의 테인트에는 표시되지 앟는 등호(=)는 무시하라.  
> kubectl은 테인트/톨러레이션 값이 null일 때 테인트와 톨러레이션을 분명히 다르게 표시한다. 

#### 테인트 효과 이해하기 

kube-proxy 파드의 다른 두 가지 톨러레이션은 준비되지 않았거나 도달할 수 없는 노드에서 파드를 얼마나 오래 실행할 수 있는지를 정의한다.   
이 두 톨러레이션은 NoSchedule 효과 대신 NoExecute를 참조한다. 

- NoSchedule 
- PreferNoSchedule 
- NoExecute 

### 노드에 사용자 정의 테인트 추가하기 

```shell
$ kubectl taint node node1.k8s node-type=production:NoSchedule 

$ kubectl run test --image busybox --replicas 5 -- sleep 99999

$ kubectl get po -o wide 
```

### 파드에 톨러레이션 추가 

```yaml
apiVersion: extension/v1beta1 
kind: Deployment 
metadata: 
  name: prod 
spec: 
  replicas: 5
  template: 
    spec:  
      tolerations: # 이톨러레이션은 프로덕션 노드에 파드가 스케쥴링될 수 있게 허용된다. 
      - key: node-type 
        Operator: Equal 
        value: production 
        effect: NoSchedule
```

```shell 
$ kubectl get po -o wide 
```

### 테인트와 톨러레이션의 활용 방안 이해 

노드는 하나 이상의 테인트를 가질 수 있으며, 파드는 하나 이상의 톨러레이션을 가질 수 있다. 지금까지 봤듯이, 테인트는 키와 효과만 갖고 있고, 값을 꼭 필요로 하지 않는다.  
톨러레이션은 Equal 연산자를 지정해 특정한 값을 허용하거나, Exists 연산자를 사용해 특정 테인트 키에 여러 값을 허용할 수 있다. 

#### 스케줄링에 테인트와 톨러레이션 사용하기 

테인트는 새 파드의 스케줄링을 방지하고, 선호하지 않는 노드를 정의하고, 노드에서 기존 파드를 제거하는 데에도 사용할 수 있다.  
필요에 맞게 테인트와 톨러레이션을 설정할 수 있다. 

#### 노드 실패 후 파드를 재스케줄링하기까지의 시간 설정 

```shell
$ kubectl get po prod-350605-1ph5h -o yaml 
... 
  tolerations:
  - effect: NoExecute  # 노드가 준비되지 않은 상태에서 파드는 재스케줄링되기 전에 300초 동안 기다린다. 
    key: node.alpha.kubernetes.io/notReady 
    operator: Exists 
    tolerationSeconds: 300 
  - effect: NoExecute # 도달할 수 없는 노드에도 동일하게 적용된다. 
    key: node.alpha.kubernetes.io/unreachable 
    operator: Exists 
    tolerationSeconds: 300
```

## 노드 어피니티를 사용해 파드를 특정 노드로 유인하기 

- 노드 어피니디와 노드 셀렉터 비교

초기 버전의 쿠버네티스에서 노드 어피니티 메커니즘은 파드 스펙의 노드 셀렉터 필드였다. 노드는 파드의 대상이 되고자 해당 필드에 지정된 모든 레이블을 포함 시켜야 했다. 
노드 셀렉터는 간단하고 잘 작동하지만 필요한 모든 것을 제공하지는 않는다. 그로 인해 더 강력한 메커니즘이 도입됐다.   
노드 셀렉터는 결국 사용이 중단될 것이므로, 새로운 노드 어피니티 규칙을 이해할 필요가 있다.  
노드 셀렉터와 유사하게 각 파드는 고유한 노드 어피니티 규칙을 정의할 수 있다. 이를 통해 꼭 지켜야 하는 필수 요구 사항이나 선호도를 지정할 수 있다. 
선호도를 지정하는 방식으로 쿠버네티스에서 어떤 노드가 특정한 파드를 선호한다는 것을 알려주면, 쿠버네티스는 해당 노드 중 하나에 파드를 스케줄링하려고 시도하게 된다.  
해당 노드에 스케줄링이 불가능하다면 다른 노드를 선택한다.  

- 디폴트 노드 레이블 검사 

노드 어피니티는 노드 셀렉터와 같은 방식으로 레이블을 기반으로 노드를 선택한다. 노드 어피니티를 사용하는 방법을 보기 전에 구글 쿠버네티스 엔진 클러스터에서 노드 중 하나의 레이블을 검사해서 
디폴드 노드 레이블이 무엇인지 살펴보자.  

```shell
$ kubectl describe node gke-kubia-default-pool-db274c5a-mjnf 
```

### 하드 노드 어피니티 규칙 지정 

```yaml
apiVersion: v1 
kind: Pod 
metadata: 
  name: kubia-gpu 
spec: 
  nodeSelector:  # 파드는 gpu=true라는 레이블이 있는 노드에만 스케줄링 된다.  
    gpu: "true"
```

위와 달리 노드 어피니티 규칙을 사용하는 경우 

```yaml
apiVersion: v1 
kind: Pod 
metadata:
  name: kubia-gpu 
spec: 
  affinity: 
    nodeAffinity:
      requireDuringSchedulingIgnoredDuringExecution: 
        nodeSelectorTerms:
        - matchExpressions: 
          - key: gpu
            operator: In 
            values: 
            - "true"
```
##### 긴 nodeAffinity 속성 이름 이해 

파드의 스펙 섹션에는 nodeAffinity 필드가 포함된 affinity 필드가 포함돼 있다. 이 필드에는 이름이 아주 긴 필드가 포함돼 있으므로, 먼저 그것에 초점을 맞추겠다. 

- requireDuringScheduling ... 이 필드 아래에 정의된 규칙은 파드가 노드로 스케줄링되고자 가져야 하는 레이블을 지정한다. 
- ... IngoredDuringExecution 이 필드 아래에 정의된 규칙은 노드에서 이미 실행중인 파드에는 영향을 미치지 않는다.  

현재 시점에는 어피니티가 파드 스케줄링에만 영향을 미치며, 파드가 노드에서 제거되지 않음을 알려주고 싶다. 이것이 모든 규칙이 항상 IgnoredDuringExecution으로 
끝나는 이유다. 결국 향후에는 쿠버네티스에서 RequiredDuringExecution 도 지원할 것이다. 즉, 노드에서 레이블을 제거하면 해당 레이블을 가지고 있는 노드에서 실행되던 파드가 해당 노드에서 
제거된다. 

##### nodeSelectorTerm 이해 

nodeSelectorTerms 필드와 matchExpressions 필드가 파드를 노드에 스케줄링하고자 노드의 레이블이 일치하도록 표현식을 정의하는 데 사용된다는 것을 쉽게 이해할 수 있을 것이다.  
예제에는 표현식이 하나만 있기 때문에 이해하기 쉽다. 노드에는 값이 true로 설정된 gpu 레이블이 있어야 한다. 

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_affinity_001.png)

노드 어피니티를 사용하면 스케쥴링 중에 노드의 우선순위를 지정할 수 있다.

### 파드의 스케줄링 시점에 노드 우선순위 지정 

새로 도입된 노드 어피니티 기능의 가장 큰 장점은 특정 파드를 스케줄링할 때 스케줄러가 선호할 노드를 지정할 수 있다는 것이다. 이는 
preferredDuringSchedulingIgnoredDuringExecution 필드를 통해 수행된다.  
여러 국가에 여러 데이터 센터를 갖고 있다고 상상해보자. 각 데이터 센터는 별도의 가용 영역을 나타낸다. 각 영역에는 자사용으로만 사용되는 특정 머신과 파트너 회사가 사용할 수 있는 
별도의 머신이 있다. 이제 여려분이 파드 몇 개를 배포하고자 하며, zone1 영역과 자사용으로 예약된 머신에 스케줄링되기를 원한다. 만약 해당 머신에 파드를 위한 공간이 충분하지 않거나 
머신을 스케줄링할 수 없는 다른 중요한 이유가 있는 경우 다른 영역과 파드너가 사용하는 머신에 스케줄링 돼도 상관 없다. 노드 어피니티 덕에 이것이 가능하다.  

#### 노드 레이블링  

먼저 노드에 적절한 레이블을 지정해야 한다. 각 노드에는 노드가 속한 가용 영역을 지정하는 레이블과 이를 전용 또는 공유 노드로 표시하는 레이블이 있어야 한다. 

## 파드 어피니티와 안티-어피니티를 이용해 함께 배치하기 