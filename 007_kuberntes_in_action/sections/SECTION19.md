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
      ... 
      tolerations: #이톨러레이션은 프로덕션 노드에 파드가 스케쥴링될 수 있게 허용된다. 
      - key: node-type 
        Operator: Equal 
        value: production 
        effect: NoSchedule
```

```shell 
$ kubectl get po -o wide 
```

## 노드 어피니티를 사용해 파드를 특정 노드로 유인하기
## 파드 어피니티와 안티-어피니티를 이용해 함께 배치하기 