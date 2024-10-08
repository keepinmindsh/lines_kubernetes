- [Section 18](#section-18)
  - [수평적 파드 오토스케일링](#수평적-파드-오토스케일링)
    - [오토 스케일링 프로세스 이해](#오토-스케일링-프로세스-이해)
      - [파드 메트릭 얻기](#파드-메트릭-얻기)
      - [필요한 파드 수 계산](#필요한-파드-수-계산)
      - [스케일링된 리소스의 레플리카 수 갱신](#스케일링된-리소스의-레플리카-수-갱신)
      - [전체 오토스케일링 과정 이해](#전체-오토스케일링-과정-이해)
    - [CPU 사용률 기반 스케일링](#cpu-사용률-기반-스케일링)
      - [CPU 사용량을 기반으로 HorizontalPodAutoscaler 생성](#cpu-사용량을-기반으로-horizontalpodautoscaler-생성)
      - [최초 오토 리스케일 이벤트 보기](#최초-오토-리스케일-이벤트-보기)
      - [스케일 업 일으키기](#스케일-업-일으키기)
      - [오토 스케일러가 디플로이먼트를 스케일 업하는 것 확인](#오토-스케일러가-디플로이먼트를-스케일-업하는-것-확인)
      - [최대 스케일링 비율 이해](#최대-스케일링-비율-이해)
      - [기존 HPA 오브젝트에서 목표 메트릭 값 변경](#기존-hpa-오브젝트에서-목표-메트릭-값-변경)
      - [메모리 소비량에 기반을 둔 스케일링](#메모리-소비량에-기반을-둔-스케일링)
      - [오브젝트 메트릭 유형 이해](#오브젝트-메트릭-유형-이해)
      - [레플리카를 0으로 감소](#레플리카를-0으로-감소)
  - [수직적 파드 오토스케일링](#수직적-파드-오토스케일링)
      - [리소스 요청 자동 설정](#리소스-요청-자동-설정)
      - [참고 할만한 Tips](#참고-할만한-tips)
  - [수평적 클러스터 노드 확장](#수평적-클러스터-노드-확장)
    - [클러스터 오토스케일러 소개](#클러스터-오토스케일러-소개)
      - [클라우드 인프라스트럭처에 추가 노드 요청](#클라우드-인프라스트럭처에-추가-노드-요청)
    - [클러스터 오토스케일러 활성화](#클러스터-오토스케일러-활성화)
    - [클러스터 스케일 다운 동안에 서비스 중단 제한](#클러스터-스케일-다운-동안에-서비스-중단-제한)


# Section 18

파드에서 실행되는 애플리케이션은 레플리케이션 컨트롤러, 레플리카셋, 디플로이먼트 또는 기타 확장 가능한 리소스 안에 있는 replicas 필드 값을 늘려 수동으로 확장할 수 있다.  
또한 파드는 컨테이너 리소스 요청 및 제한을 늘력 수직으로 확장하는 것도 가능하다. 스케일을 수동으로 제어하는 건 순간적인 부하를 예측할 수 있거나, 부하가 장시간에 걸쳐 점진적으로 
변화하는 경우에는 괜찮다. 하지만 갑자기 발생하는 예측할 수 없는 트래픽 증가 현상을 수동으로 개입해 처리하는 것은 이상적이지 않다.  

## 수평적 파드 오토스케일링

수평적 파드 오토스케일링은 컨트롤러가 관리 파드의 레플리카 수를 자동으로 조정하는 것을 말한다. 이것은 Horizontal 컨트롤러에 의해 수행되며,  
HorizontalPodAutoscaler 리소스를 작성해 활성화시키고 원하는 대로 설정한다. 컨트롤러는 주기적으로 파드 메트릭을 확인해,  
HorizontalPodAutoScaler 리소스에 설정돼 있는 대상 메트릭 값을 만족하는 레플리카 수를 계산한다. 그리고 대상 리소스 안에 있는 replicas 필드 값을 조정한다.  

### 오토 스케일링 프로세스 이해 

- 확장 가능한 리소스 오브젝트에서 관리하는 모든 파드의 메트릭을 가져온다. 
- 메트릭을 지정한 목표 값과 같거나 가깝도록 하기 위해 필요한 파드 수를 계산한다.  
- 확장 가능한 리소스의 replicas 필드를 갱신한다. 

#### 파드 메트릭 얻기 

오토 스케일러는 파드 메트릭을 수집하지 않고, 다른 소스에 메트릭을 가져온다.  
파드와 노드 메트릭은 모든 노드에서 실행되는 kubelet에서 실행되는 cAdvisor 에이전트에 의해 수집된다. 수집한 메트릭은 클러스터 전역 구성 요소인  
힙스터에 의해 집계된다. 수집한 메트릭은 클러스터 전역 구성 요소인 힙스터에 의해 집계된다. 수평적 파드 오토스케일러 컨트롤러는 힙스터에 REST를 통해  
질의해 모든 파드의 메트릭을 가져온다. 

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_autoscaling_001.png)

#### 필요한 파드 수 계산  

오토스케일러의 스케일링 대상이 되는 리소스에 속해 있는 파드의 모든 메트릭을 가지고 있으면, 이 메트릭을 사용해 필요한 레플리카 수를 파악할 수 있다.  
모든 레플리카에서 메트릭의 평균 값을 이용해 지정한 목표 값과 가능한 가깝게 하는 숫자를 찾아야 한다. 이 계산의 입력은 파드 메트릭 세트이고,  
출력은 하나의 정수이다.  

오토 스케일러가 단일 메트릭만을 고려하도록 설정돼 있다면 필요한 레플리카 수를 계산하는 것은 간단하다. 모든 파드의 메트릭 값을 더한 뒤  
HorizontalPodAutoscaler 리소스에 정의된 목표 값으로 나눈 값을 그 다음으로 큰 정수로 반올힘해서 구한다. 실제 계산은 메트릭 값이   
불안정한 상태에서 빠르게 변할 때 오토스케일러가 같이 요동치지 않도록 하기 때문에 이보다 더 복잡하다.   

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_autoscaling_002.png)

오토스케일링 여러 파드 메트릭을 기반으로 하는 경우 계산이 그렇게 복잡하지는 않다. 오토스케일러는 각 메트릭의 레플리카 수를 개별적으로 계산한 뒤 
가장 높은 값을 취한다.  

#### 스케일링된 리소스의 레플리카 수 갱신 

오토스켈링 작업의 마지막 단계는 스케일링 된 리소스 오브젝트 레플리카 개수 필드를 원하는 값으로 갱신해, 레플리카셋 컨트롤러가 추가 파드를 시작하거나 초과한 파드를 삭제하도록 하는 것이다.  
오토스케일러 컨트롤러는 스케일 대상 리소스의 replicas 필드를 스케일 서브 리소스를 통해 변경된다. 이는 스케일 서브 리소스를 통해 노출되는 것을 제외하고, 오토스케일러가 리소스의  
세부 사항을 알 필요 없이 수행할 수 있게 해준다. 

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_autoscaling_003.png)

- 디플로이먼트 
- 레플리카셋 
- 레플리케이션 컨트롤러 
- 스테이트풀셋 

#### 전체 오토스케일링 과정 이해 

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_autoscaling_004.png)

파드에서 cAdvisor로 이어지는 화살표는 힙스터를 지나 수평적 파드 오토스케일러에서 끝난다. 이 화살표는 메트릭 데이터의 흐름을 나타낸다.  
여기서 중요한 점은, 각 구성요소는 메트릭을 다른 구성 요소에서 주기적으로 가져온다는 것이다.  결과적으로 메트릭 데이터가 전파돼 재조정 작업이  
수행되기까지는 시간이 걸린다. 

### CPU 사용률 기반 스케일링

오토스케일링 기반이 되는 가장 중요한 메트릭으로는 파드 내부 프로세스가 소비하는 CPU 사용량일 것이다.  
서비스를 제공하는 파드가 몇 개 있다고 가정하자. 
CPU 사용량이 100%에 도달하면 더 이상 요구에 대응할 수 없어 스케일 업이나 스케일 아웃이 필요하다. 
여기서 우리는 수평적 파드 오토스케일러만 얘기하고 있기 때문에 스케일 아웃에 관해서만 집중해보자.  
스케일 아웃이 이루어지면 평균 CPU 사용량은 줄어들 것이다.  

--- 

오토스케일러에 한해서는 파드의 CPU 사용률을 결정할 때 파드가 보장받은 CPU 사용량만이 중요하다. 오토 스케일러는 파드의 실제 CPU 사용량과 CPU 요청을 비교하는데  
이는 오토스케일링이 필요한 파드는 직접 또는 간접적으로 LimitRange 오브젝트를 통해 CPU 요청을 설정해야 한다는 것을 의미한다.  

> 항상 목표 CPU 사용량을 100%에 훨씬 못미치게 설정해, 갑자기 순간적인 부하가 발생하는 것을 제어하는데 필요한 공간을 확보해야 한다. 

#### CPU 사용량을 기반으로 HorizontalPodAutoscaler 생성 

```yaml
apiVersion: extensions/v1beta1 
kind: Deployment 
metadata: 
  name: kubia 
spec:
  replicas: 3 
  template: 
    metadata: 
      name: kubia 
      labels: 
        app: kubia 
    spec: 
      containers:
      - image: luksa/kubia:v1 
        name: nodejs 
        resources: 
          requests: 
            cpu: 100m 
```

```shell
$ kubectl autoscale deployment kubia --cpu-percent=30 --min=1 --max=5 
```

HPA 오브젝트를 생성하고 kubia 디플로이먼트를 스케일링 대상으로 설정한다. 파드의 목표 CPU 사용률을 30%로 지정하고  
최소 및 최대 레플리카 수를 지정한다. 오토스케일러는 CPU 사용률을 30%대로 유지하기 위해 레플리카 수를 지정한다.  
오토스케일러는 CPU 사용륭을 30%대로 유지하기 위해 레플리카 수를 계속 저장하지만 1개 미만으로 줄이거나 5개를 초과하는 레플리카를 만들지는 않는다. 

> 항상 레플리카셋이 아닌 디플로이먼트를 오토스케일 대상으로 해야 한다.  
> 이렇게 하면 애플리케이션 업데이트 시에도 원하는 레플리카 수를 계속 유지할 수 있다. 
> 수동 스케일링에도 동일한 규칙이 적용된다. 

```yaml
$ kubectl get hpa.v2beta1.autoscaling kubia -o yaml 
```

#### 최초 오토 리스케일 이벤트 보기 

```shell
$ kubectl get hpa 

$ kubectl get deployment 

$ kubectl describe hpa 
```

#### 스케일 업 일으키기 

```shell
$ kubectl expose deployment kubia --port=80 --target-port=8080 
```

```shell
# 동시에 여러 시소스 관찰하기 
$ watch -n 1 kubectl get hpa,deployment 
```

> kubectl get 명령에 여러 리소스 유형을 쉼표로 구분해 나열한다. 

```shell
$ kubectl run -it  --rm --restart=Never loadgenerator --image=busybox
```

-it 옵션을 kubectl exec 명령을 실행 할 때 몇번 본적 있다. 이 명령은 kubectl run 명령을 실행할 때도 사용할 수 있다.  
이 옵션은 콘솔을 대상 프로세스에 붙여, 프로세스의 출력을 직접 볼 수 있게 해줄 뿐만 아니라 CTRL+C를 눌러 프로세스를 종료할 수도 있다.  
--rm 옵션은 파드가 종료된 후에 삭제하도록 하고 --restart=Never 옵션은 kubectl run 명령이 디플로이먼트 오브젝트를 통해 관리하는 대신, 
관리되지 않는 파드를 직접 만들도록 한다. 이 옵션 조합은 클러스터 안에서 이미 존재하는 파드에 도움을 받지 ㅇ낳고 명령을 실행하는 데 유용하다.  
원하는 명령을 로컬에서 실행하는 것과 동일하게 수행되며, 명령이 종료되면 모든 것을 깨끗하게 정리한다. 

#### 오토 스케일러가 디플로이먼트를 스케일 업하는 것 확인 

처음에는 레플리카 한 개가 요청을 처리하면서 CPU 사용량이 108 %로 급증했다. 108을 30으로 나누면 3.6을 얻게 되는데, 
오토 스케일러는 이 값을 반올림해서 4로 만든다. 108을 30으로 나누면 3.6을 얻게되는데, 오토 스케일러는 이 값을 반올림해서 4로 만든다. 108을 4오 나누면 27%를 얻는다. 만약 
오토 스케일러가 네 개의 파드로 스케일 업하면, 파드들의 평균 CPU 사용률은 27% 근처일 것으로 예상되고, 이 값은 목표 값이 30%에 가깝고 관찰된 CPU 사용률과 거의 비슷하다.  

#### 최대 스케일링 비율 이해 

CPU 사용율은 최대 108% 까지 증가 했지만 일반적으로 초기 CPU 사용량은 더 크게 증가할 수 있다. 초기 평균 CPU 사용률이 더 높으면, 30% 목표를 달성하기 위해  
다섯 개의 레플리카가 필요하지만 오토스케일러는 여전히 네 개의 파드까지만 확장한다. 한 번의 확장 단계에서 추가할 수 있는 레플리카의 수는 제한돼 있끼 때문이다. 오토 스케일러는 
두 개를 초과하는 레플리카가 존재할 경우 한 번의 수행에서 최대 두 배의 레플리카를 생성할 수 있다. 만약 한 개 또는 두 개의 레플리카가 존재하면 한 단계에서 최대 네 개의 레플리카까지 
확장할 수 있다.  
 또한 이전 작업 이후에 이어지는 오토 스케일 작업이 얼마나 빨리 일어날 수 있는 지에 대한 제한도 있다. 현재는 지난 3분 동안 아무런 리스케일링 이벤트가 발생하지 않을 경우에만  
스케일 업이 일어난다. 스케일 다운 이벤트는 5분 간격으로 더 적제 일어난다. 이 제한을 기억한다면 메트릭은 병백히 리스케일 작업이 필요하다고 가리키지만, 오토스케일러가 리스케일 작업을 거부한 이유가 
궁금하지 않을 것이다. 

#### 기존 HPA 오브젝트에서 목표 메트릭 값 변경 

```yaml
spec:
  maxReplicas: 5
  metrics:
  - resources: 
      name: cpu
      targetAverageUtilization: 60 # 이 값을 30에서 60으로 변경한다. 
```

대부분의 다른 리소스와 마찬가지로 리소스를 수정한 후에 변경 사항은 오토스케일러 컨트롤러에 의해 감지돼 동작한다.  
그리고 HPA 리소스를 삭제하고 다른 목표 값으로 다시 생성하는 것도 할 수 있다. HPA 리소스를 삭제하는 것은 대상 리소스의  
오토 스케일링을 비활성화하는 것이기 때문에 그 때 크기로 계속 유지된다. 

#### 메모리 소비량에 기반을 둔 스케일링 

메모리 기반 오토스케일링은 CPU 기반 오토스케일링에 비해 훨씬 문제가 많다.  
가장 큰 이유는 스케일 업 후에 오래된 파드는 어떻게는 메모리를 해제하는 것이 필요하기 때문이다. 해제하는 작업은 애플리케이션이 직접해야하며 시스템이  
할 수 있는 것이 아니다. 시스템이 할 수 있는 것은 이전보다 작음 메모리를 사용하기를 기대하면서 애플리케이션을 
종료하고 다시 시작하는 것뿐이다. 

#### 오브젝트 메트릭 유형 이해 

Object 메트릭 유형은 오토 스케일러가 파드를 확장할 때 파드에 직접 관리되지 않는 메트릭을 기반으로 하도록 만든다. 
예를 들어 파드를 확장할 때 인그레스 오브젝트 같은 클러스터 오브젝트 메트릭을 이용할 수 있다. 

```yaml
... 
spec:
  metrics: # 특정 오브젝트 메트릭 사용 
  - type: Object 
    resource: 
      metricName: latencyMillis # 메트릭 이름 
      target: # 오토 스케일러가 메트릭을 얻어올 오브젝트 정리 
        apiVersion: extentions/v1beta1 
        kind: Ingress 
        name: frontend
      targetValue: 20 # 오토스케일러는 메트릭이 이 값에 가까이 유지되도록 확장한다. 
      scaleTargetRef: 
        apiVersion: extensions/v1bet1 
        kind: Deployment 
        name: kubia 
... 
```

#### 레플리카를 0으로 감소 

- 유휴, 유해제 
  - 
  - 이는 특정 서비스를 제공하는 파드를 0으로 축소 할 수 있게 해준다. 새로운 요청이 들어오면 파드가 깨어나 요청을 처리할 수 있을 때까지 차단돼 있다가 이후에 요청이 파드로 전달 된다. 

## 수직적 파드 오토스케일링

수평적 파드 확장은 훌륭하지만 모든 애플리케이션이 수평적으로 확장 가능한 것은 아니다. 수평적 확장이 불가능한 애플리케이션의 경우 유일한 옵션은 수직적으로 확장하는 것이다. 
노드는 일반적으로 하나의 파드 요청 보다는 많은 리소스를 갖고 있기 때문에 거의 대부분은 파드를 수직적으로 확장할 수 있어야 한다. 

#### 리소스 요청 자동 설정 

InitialResources 라는 어드미션 컨트롤 플러그인에 의해 제공된다. 새로운 파드가 리소스 요청 없이 생성되면 플러그인은 파드 컨테이너의 과거 리소스 사용량 데이터를 살펴보고 요청 값을 적잘하게 설정한다. 
리소스 요청 값을 지정하지 않고 파드를 배포한 뒤 쿠버네티스에 의존해 컨테이너의 리소스 요구 사항을 파악할 수도 있다. 쿠버네티스는 효과적으로 파드의 수직적 확장을 수행한다.

#### 참고 할만한 Tips 

> [https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler)  
> [https://cloud.google.com/kubernetes-engine/docs/concepts/verticalpodautoscaler?hl=ko](https://cloud.google.com/kubernetes-engine/docs/concepts/verticalpodautoscaler?hl=ko)

## 수평적 클러스터 노드 확장 

수평적 파드 오토스케일러는 필요할 때 추가 파드 인스턴스를 생성한다. 

### 클러스터 오토스케일러 소개 

클러스터 오토 스케일러는 노드에 리소스가 부족해서 스케줄링할 수 없는 파드를 발견하면 추가 노드를 자동으로 공급한다. 또한 오랜 시간 동안 사용률이 낮으면 노드를 줄인다.  

#### 클라우드 인프라스트럭처에 추가 노드 요청 

새 파드가 생성된 후 스케줄러가 기존 노드에 스케줄링 할 수 없는 경우, 새로운 노드가 공급된다. 클러스터 오토스케일러는 이런 파드를 갖고 클라우드 제공자에 추가 노드를  
시작하도록 요청한다. 그러나 이를 수행하기 전에 새로운 노드가 파드를 수용할 수 있는지 확인한다. 만약에 그렇지 않으면 새 노드를 시작하는게 의미가 없다.  

클라우드 제공자는 동일한 크기의 노드들을 그룹으로 묶는다. 클러스터 오토스케일러는 단순히 "추가 노드를 주세요"라고 말할 수 없고, 노드 유형을 지정해야 한다.  

클러스터 오토스케일러는 사용 가능한 노드 그룹을 검사해 최소한 하나의 노드 유형이 스케줄링되지 않은 파드를 수용할 수 있는지 확인한다.  
이런 노드 그룹이 정확히 하나만 있으면, 클라우드 제공자가 새로운 노드를 그룹에 추가하도록 오토스케일러는 해당 노드 그룹의 크기를 증가 시킨다.  
둘 이상의 옵션을 사용할 수 있는 경우 오토스케일러는 가장 좋은 옵션을 선택한다. "좋은"의 정확한 의미는 설정 가능하다는 것을 의미한다.  
최악의 경우 무작위로 하나를 선택한다. 스케줄링되지 못한 파드에 오토스케일러가 반응하는 간당한 개요를 보자 

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_autoscaler_001.png)

- 노드 종료

또한 클러스터 오토스케일러는 노드 리소스가 충분히 활용되고 있지 않을 때 노드 수를 줄여야 한다.  
오토 스케일러는 모든 노드에 요청된 CPU와 메모리를 모니터링해 이를 수행한다. 특정 노드에서 실행 중인 모든 파드의 CPU와 메모리 요청이 50% 미만이면  
해당 노드는 필요하지 않은 것으로 간주한다. 

### 클러스터 오토스케일러 활성화 

- 구글 쿠버네티스 엔진 
- 구글 컴퓨트 엔진 
- 아마존 웹 서비스 
- 마이크로소프트 애저 

오토스케일러를 시작하는 방법은 어디에서 쿠버네티스 클러스터가 실행 중인가에 따라 다른다.  
kubia 클러스터가 GKE 위에서 동작하고 있다면, 클러스터 오토스케일러를 다음 명령으로 활성화 될 수 있다.  

```shell
$ gcloud container clusters update kubia --enable-autoscaling --min-nodes=3 --max-nodes=5 
```

### 클러스터 스케일 다운 동안에 서비스 중단 제한 

노드가 예기치 않게 실패할 경우 파드가 사용 불가 상태가 되는 것을 막을 수 있는 방법이 없다. 하지만 노드 종료가 클러스터 오토스케일러나 시스템 관리자로 인해 이뤄지는 상황이라면,  
추가 기능을 통해 해당 노드에서 실행되는 파드가 제공하는 서비스를 중단되지 않도록 할 수 있다.   

특정 서비스에서는 최소 개수의 파드가 항상 실행돼야 한다. 이는 쿼럼 기반 클러스터 애플리케이션인 경우 특히 그렇다.  
이런 이유로 쿠버네티스는 스케일 다운 등의 작업을 수행할 경우에도 유지돼야 하는 최소 파드 개수를 지정하는 방법을 제공한다.  
Pod DisruptionBudget 리소스를 만들어 이를 수행할 수 있다.  

리소스 이름이 복잡해 보이지만, 가장 간단하게 사용할 수 있는 쿠버네티스 리소스 중 하나다. 오직 파드 레이블 셀렉터와 항상 사용 가능해야 하는 파드의 최소 개수 혹은 파드의 
최대 개수를 정의할 수 있다. 

```shell
# kubia 파드가 항상 세 개 이상의 인스턴스를 갖도록 하려면, PodDistruptionbudget 리소스를 만든다. 
$ kubectl create pdb kubia-pdb --selector=app=kubia --min-available=3
```

```shell
$ kubectl get pdb kubia-pdb -o yaml 
```