- [Section 5 - 파드를 안정적으로 유지하기 그리고 레플리카셋](#section-5---파드를-안정적으로-유지하기-그리고-레플리카셋)
  - [Cheat Sheet](#cheat-sheet)
  - [파드를 안정적으로 유지하기](#파드를-안정적으로-유지하기)
    - [라이브니스 프로브](#라이브니스-프로브)
  - [레플리카셋](#레플리카셋)
    - [레플리카셋과 리플리케이션 컨트롤러 비교](#레플리카셋과-리플리케이션-컨트롤러-비교)
    - [리플리카셋 생성, 검사](#리플리카셋-생성-검사)
    - [레플리카셋의 표현적인 레이블 셀렉터 사용하기](#레플리카셋의-표현적인-레이블-셀렉터-사용하기)
    - [레플리카셋 정리](#레플리카셋-정리)
    - [레플리카 셋을 사용하는 시기](#레플리카-셋을-사용하는-시기)
    - [파드 삭제 비용](#파드-삭제-비용)
    - [Replica Set 의 스케일링](#replica-set-의-스케일링)
    - [Replica Set 설정시 주의해야할 사항](#replica-set-설정시-주의해야할-사항)
    - [레플리카 셋의 대안](#레플리카-셋의-대안)
      - [디플로이먼트 권장](#디플로이먼트-권장)
      - [잡](#잡)
      - [데몬셋](#데몬셋)
    - [ReplicaSet 의 다양한 사례](#replicaset-의-다양한-사례)
  - [데몬셋](#데몬셋-1)
    - [데몬셋을 사용해 각 노드에서 정확히 한개의 파드 실행](#데몬셋을-사용해-각-노드에서-정확히-한개의-파드-실행)
    - [데몬셋으로 모든 노드에 파드 실행하기](#데몬셋으로-모든-노드에-파드-실행하기)
    - [예제를 사용한 데몬셋 설명](#예제를-사용한-데몬셋-설명)
    - [좀더 자세항 사양 작성 ( 데몬 셋 )](#좀더-자세항-사양-작성--데몬-셋-)
    - [필요한 레이블을 노드에 추가하기](#필요한-레이블을-노드에-추가하기)
  - [완료 가능한 단일 태스크를 수행하는 파드 실행](#완료-가능한-단일-태스크를-수행하는-파드-실행)
    - [잡 리소스 소개](#잡-리소스-소개)
    - [잡 리소스 정의](#잡-리소스-정의)
    - [파드를 실행한 잡 보기](#파드를-실행한-잡-보기)
    - [잡에서 여러 파드 실행하기](#잡에서-여러-파드-실행하기)
    - [잡스케일링](#잡스케일링)
    - [잡 파드가 완료되는데 걸리는 시간 제한하기](#잡-파드가-완료되는데-걸리는-시간-제한하기)
    - [잡을 주기적으로 또는 한 번 실행되도록 스케쥴링하기](#잡을-주기적으로-또는-한-번-실행되도록-스케쥴링하기)
  - [References](#references)


# Section 5 - 파드를 안정적으로 유지하기 그리고 레플리카셋

## Cheat Sheet 

```shell
$ kubectl autoscale deployment foo --min=2 --max=10

$ kubectl autoscale rc foo --max=5 --cpu-percent=80

$ kubectl scale --replicas=3 rs/foo 

$ kubectl scale --replicas=3 -f foo.yaml

$ kubectl scale --current-replicas=2 --replicas=3 deployment/mysql 

$ kubectl scale --replicas=5 rc/foo rc/bar rc/baz 

$ kubectl scale --replicas=3 statefulset/web
```

## 파드를 안정적으로 유지하기 

### 라이브니스 프로브 

쿠버네티스는 라이브니스 프로브를 통해서 컨테이너가 살아있는지 확인할 수 있다. 

> 애플리케이션 시작시간을 고려해 초기 지연을 설정해야 한다는 점을 명심할 것! 

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: registry.k8s.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```

라이브니스 프로브를 위해 특정 URL 경로에 요청하도록 프로브를 구성해 애플리케이션 내에서 실행 중인 모든 주요 구성 요소가 살아 있는지 
또는 응답이 없는지 확인하도록 구성할 수 있다. 

> HTTP 엔드 포인트에 인증이 필요하지 않는지 확인하라. 그렇지 않으면 프로브가 항상 실패해 컨테이너가 무한정으로 재시작된다. 

## 레플리카셋

k8s 에서 사용하는 가장 작은 단위가 바로 Pod 입니다. 이 Pod 하나로 서비스를 올린다면, HA(고가용성)을 보장하지 못하기 때문에  
Pod를 복제하여 Pod 의 FailOver 시에 자동적으로 다른 Pod로 서빙해주어야 합니다. 그것을 k8s 에서 Object Controller 형식으로 
지원해주는 것이 바로 ReplicaSet 입니다. 

- Selector 는 **라벨을 기반**으로 하며, 라벨을 지정하면 해당 라벨을 가신 **모든 Pod를 관리**합니다.  
- Selector 는 **집합 단위**로 사용됩니다. 즉, **in, notin, exists** 같은 연산자를 사용 가능합니다. 

### 레플리카셋과 리플리케이션 컨트롤러 비교

레플리카 셋은 레플리케이션 컨트롤러와 똑같이 동작하지만 좀 더 풍부한 표현식을 사용하는 파드 셀렉터를 갖고 있다. 레플리케이션컨터롤러의 레이블 셀렉터는 특정 레이블이 있는 파드만
매칭 시킬 수 있는 방면, 레플리카셋의 셀렉터는 특정 레이블이 없는 파드나 레이블의 값과 상관 없이 특정 레이블의 키를 갖는 파드를 매칭 시킬 수 있다.

예를들어 차이점을 정리해보면

- 레플리케이션 컨트롤러
    - 하나의 레플리케이션 컨트롤러는 레이블이 env=production 인 파드와 레이블이 env=devel를 매칭 시킬수 있다.
    - 레플리케이션 컨트롤러는 레이블의 키로 파드를 매칭시킬 수 없다.
- 레플리카셋
    - 레플리카셋은 하나의 레플리카셋으로 두 파드 세트 모두를 매칭시켜 하나의 그룹으로 취급할 수 있다.
    - 레플리카셋은 레이블의 키값으로 파드를 매칭시킬 수 있다.
        - 예) env 키가 있는 레이블을 갖는 모든 파드를 매핑 시킬 수 있다. ( env=* )


```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # 케이스에 따라 레플리카를 수정한다.
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
        - name: php-redis
          image: gcr.io/google_samples/gb-frontend:v3
```

- selector ~ : 여기서는 레플리케이션컨트롤러와 유사한 간단한 matchLabels 셀렉터를 사용한다.
- template ~ : 템플릿은 레플리케이션 컨트롤러와 동일하다.

```shell
$ kubectl describe rs/frontend

$ kubectl get pods

$ kubectl get pods frontend-b2zdv -o yaml
```

> API 버전 속성에 대해서
> - API 그룹 ( apps )
> - API 버전 ( v1beta2 )
    > core API 그룹에 속하는 어떤 쿠버네티스 리소스들은 apiVersion 필드를 지정할 필요가 없다는 것을 책 전체에 걸쳐서 보게 될 것이다.
    > 최신 쿠버네티스 버전에서 도입된 다른 리소스는 여러 API 그룹으로 분류된다.

### 리플리카셋 생성, 검사

레플리카셋은 모든 쿠버네티스 API 오브젝트와 마찬가지로 apiVersion, kind, metadata 필드가 필요하다. 레플리카셋에 대한 kind 필드의 값은 항상 레플리카셋이다.

- replicaset 조회

```shell
$ k get rs
NAME                                       DESIRED   CURRENT   READY   AGE
lines-admin-nextjs-deployment-5f85b84f87   0         0         0       58d
lines-admin-nextjs-deployment-864f94fc8d   2         2         2       3d14h
```

- 상세 정보 확인하기

```shell
$ k describe rs
Name:           lines-admin-nextjs-deployment-5f85b84f87
Namespace:      default
Selector:       app=lines-admin-nextjs,pod-template-hash=5f85b84f87
Labels:         app=lines-admin-nextjs
                pod-template-hash=5f85b84f87
Annotations:    deployment.kubernetes.io/desired-replicas: 2
                deployment.kubernetes.io/max-replicas: 3
                deployment.kubernetes.io/revision: 1
                lines/admin-annotation: dream come true
                lines/hello-annotation: dream come true
Controlled By:  Deployment/lines-admin-nextjs-deployment
Replicas:       0 current / 0 desired
Pods Status:    0 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=lines-admin-nextjs
           pod-template-hash=5f85b84f87
  Containers:
   lines-admin-nextjs:
    Image:      gcr.io/lines-infra/lines_admin_front:v0.1.0
    Port:       3000/TCP
    Host Port:  0/TCP
    Limits:
      cpu:     1024m
      memory:  1Gi
    Requests:
      cpu:        100m
      memory:     32Mi
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:           <none>
```

### 레플리카셋의 표현적인 레이블 셀렉터 사용하기

```yaml
selector: 
  matchExpressions: 
    - key: app 
      operator: In 
      values: 
        - kubia 
```

- In은 레이블의 값이 지정된 값 중 하나와 일치해야 한다.
- NotIn은 레이블의 값이 지정된 값과 일치하지 않아야 한다.
- Exists는 파드는 지정된 키를 가진 레이블이 포함돼야한다. 이 연산자를 사용할 때는 값 필드를 지정하지 않아야 한다.
- DoesNotExist는 파드에 지정된 키를 가진 레이블이 포함돼 있지 않아야 한다. 값 필드를 지정하지 않아야 한다.

### 레플리카셋 정리

```yaml
$ k delete rs kubia 
```

### 레플리카 셋을 사용하는 시기 

레플리카셋은 지정된 수의 파드 레플리카가 항상 실행되도록 보장한다. 그러나 디플로이먼트는 레플리카셋을 관리하고 다른 유용한 기능과 함께 
파드에 대한 선언적 업데이트를 제공하는 상위개념이다. 

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # 케이스에 따라 레플리카를 수정한다.
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
```

### 파드 삭제 비용 

[https://kubernetes.io/ko/docs/reference/labels-annotations-taints/#pod-deletion-cost](https://kubernetes.io/ko/docs/reference/labels-annotations-taints/#pod-deletion-cost) 어노테이션을 이용하여, 
레플리카 셋을 스케일 다운 할 때 어떤 파드부터 먼저 삭제할 지에 대한 우선순위를 설정할 수 있다.  

이 어노테이션은 파드에 설정되어야 하며, [-2147483647, 2147483647] 범위를 갖는다.   
이 어노테이션은 하나의 레플리카셋에 있는 다른 파드와의 상대적 삭제 비용을 나타낸다.   
삭제 비용이 낮은 파드는 삭제 비용이 높은 파드보다 삭제 우선순위가 높다.

### Replica Set 의 스케일링  

- Pending 상태인 (+ 스케줄링할 수 없는) 파드가 먼저 스케일 다운된다.
- controller.kubernetes.io/pod-deletion-cost 어노테이션이 설정되어 있는 파드에 대해서는, 낮은 값을 갖는 파드가 먼저 스케일 다운된다.
- 더 많은 레플리카가 있는 노드의 파드가 더 적은 레플리카가 있는 노드의 파드보다 먼저 스케일 다운된다.
- 파드 생성 시간이 다르면, 더 최근에 생성된 파드가 이전에 생성된 파드보다 먼저 스케일 다운된다. (LogarithmicScaleDown 기능 게이트가 활성화되어 있으면 생성 시간이 정수 로그 스케일로 버킷화된다)

### Replica Set 설정시 주의해야할 사항 

단독 파드를 생성하는 것에는 문제가 없지만, 단독 파드가 레플리카셋의 셀렉터와 일치하는 레이블을 가지지 않도록 
하는 것을 강력하게 권장한다. 그 이유는 레플리카셋이 소유하는 파드가 템플릿에 명시된 파드에만 국한되지 않고,  
이전 섹션에서 명시된 방식에 의해서도 다른 파드의 획득이 가능하기 때문이다.  

이와 같은 경우에는 

- 처음 동작 

```shell
$ kubectl get pods
NAME             READY   STATUS        RESTARTS   AGE
frontend-b2zdv   1/1     Running       0          10m
frontend-vcmts   1/1     Running       0          10m
frontend-wtsmm   1/1     Running       0          10m
pod1             0/1     Terminating   0          1s
pod2             0/1     Terminating   0          1s
```

- 최종 결과 

```shell
$ kubectl get pods
NAME             READY   STATUS    RESTARTS   AGE
frontend-hmmj2   1/1     Running   0          9s
pod1             1/1     Running   0          36s
pod2             1/1     Running   0          36s
```

### 레플리카 셋의 대안 

#### 디플로이먼트 권장 

디플로이먼트는 레플리카 셋을 소유하거나 업데이트를 하고, 파드의 선언적인 업데이트와 서버측 롤링 업데이트를 할 수 있는 오브젝트이다.  
디플로이먼트는 레플리카 셋을 소유하거나 관리한다. 따라서 레플리카 셋을 원한다면 디플로이먼트를 사용하는 것을 권장한다.  

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

#### 잡 

스스로 종료되는 것이 예상되는 파드의 경우에는 레플리카셋 대신 잡을 이용한다. 

#### 데몬셋 

머신 모니터링 및 머신 로깅과 같은 머신 레벨의 기능을 제공하는 파드를 위해서는 레플리카셋 대신 데몬셋을 사용한다. 

### ReplicaSet 의 다양한 사례 

- ReplicaSet 

```yaml
apiVersion: apps/v1 
kind: ReplicaSet   
Metadata: 
  name: some-name
  labels:
    app: some-App
    tier: some-Tier
Spec: 
  replicas: 3 # Here we tell k8s how many replicas we want
  Selector: # A label selector field. 
    matchLabels:
      tier: some-Tier
    matchExpressions:
      - {key: tier, operator: In, values: [some-Tier]} #set-based operators
  template:
    metadata:
      labels:
        app: some-App
        tier: someTier
    Spec: 
      Containers:
```

- Deployment 

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

## 데몬셋

### 데몬셋을 사용해 각 노드에서 정확히 한개의 파드 실행

클러스터의 모든 노드에, 노드당 하나의 파드가 실행되길 원하는 경우가 있을 수 있다. 노드가 클러스터에 추가되면 파드도 추가된다.  

예) 로그수집기, 리소스 모니터, Kube-proxy 프로세스 등등  

- 모든 노드에서 클러스터 스토리지 데몬 실행 
- 모든 노드에서 로그 수집 데몬 실행 
- 모든 노드에서 노드 모니터링 데몬 실행 

### 데몬셋으로 모든 노드에 파드 실행하기

레플리카셋이 클러스터에 원하는 수의 파드 복제본이 존재하는지 확인하는 반만, 데몬셋에는 원하는 복제본 수라는 개념이 없다.
파드 셀렉터와 일치하는 파드 하나가 각 노드에서 실행중인지 확인하는 것이 데몬셋이 수행해야하는 역할이기 때문에 복제본의 개념이 필요하지 않다.

### 예제를 사용한 데몬셋 설명

SSD를 갖는 모든 노드에서 실행돼야하는 ssd-monitor 라는 데몬이 있다고 가정하자. SSD를 갖고 있다고 표시된 모든 노드에서 이 데몬을 실행하는 데몬셋을 만든다.
클러스터 관리자가 이런 모든 노드에 disk=ssd 레이블을 추가했으므로 해당 레이블이 있는 노드만 선택하는 노드 셀렉터를 사용해 데몬셋을 작성한다.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ssd-monitor
spec:
  selector:
    matchLabels:
      app: ssd-monitor
  template:
    metadata:
      labels:
        app: ssd-monitor
    spec:
      nodeSelector:
        app: sdd-monitor
      containers:
        - name: main
          image: luksa/ssd-monitor
```

```shell
k create -f ~/sources/02_linesgits/lines_kubernetes/007_kuberntes_in_action/p191_deamonsets/deamon_sets_sample.yaml
```

### 좀더 자세항 사양 작성 ( 데몬 셋 ) 

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      # 이 톨러레이션(toleration)은 데몬셋이 컨트롤 플레인 노드에서 실행될 수 있도록 만든다.
      # 컨트롤 플레인 노드가 이 파드를 실행해서는 안 되는 경우, 이 톨러레이션을 제거한다.
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```

### 필요한 레이블을 노드에 추가하기

```shell
k get node 

k label node {node name} disk=ssd

k get po   
```

데몬셋을 삭제하거나 대상으로 타케팅된 레이블이 변경될 경우 daemonSet에 의해서 생성된 파드도 삭제된다.

> [Update Daemon Set](https://kubernetes.io/ko/docs/tasks/manage-daemon/update-daemon-set/)   
> [Rollback Daemon Set](https://kubernetes.io/ko/docs/tasks/manage-daemon/rollback-daemon-set/)   
> [Running Pod On Some Nodes](https://kubernetes.io/docs/tasks/manage-daemon/pods-some-nodes/)  

## 완료 가능한 단일 태스크를 수행하는 파드 실행

### 잡 리소스 소개

잡은 작업이 제대로 완료되는 것이 중요한 임시 작업에 유용하다. 관리되지 않는 파드에서 작업을 실행하고 완료될 때까지 기다릴 수 있지만 작업이 수행되는 동안
노드에 장애가 발생하거나 파드가 노드에서 제거되는 경우 수동으로 다시 생성해야 한다. 완전히 잡을 완료하는데 몇 시간이 걸리는 경우 이 작업을 수행으로 수행한다는
것은 말이 안되는 일이다.

### 잡 리소스 정의

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
        - name: pi
          image: perl:5.34.0
          command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4

```

파드 스펙에서는 컨테이너에서는 실행 중인 프로세스가 종료될 때 쿠버네티스가 수행할 작업을 지정할 수 있다. 이 작업 파드 스펙 속성인 restartPolicy 로 수행하며
기본 값은 Always 이다. 잡 파드는 무한정 실행하지 않으므로 기본정책을 사용할 수 있다. 따라서 restartPolicy를 OnFailure나 Never로 명시적으로 설정해야한다.

### 파드를 실행한 잡 보기

```shell
$ k get jobs 

$ k get po 
NAME                                             READY   STATUS              RESTARTS   AGE
lines-admin-nextjs-deployment-864f94fc8d-cf65q   1/1     Running             0          3d15h
lines-admin-nextjs-deployment-864f94fc8d-lmtgc   1/1     Running             0          3d15h
pi-gwdzd                                         0/1     ContainerCreating   0          11s

$ k logs pi-gwdzd
3.1415926535897932384626433832795028841971693993751058209749445923078164062862089986280348253421170679821480865132823066470938446095505822317253594081284811174502841027019385211055596446229489549303819644288109756659334461284756482337867831652712019091456485669234603486104543266482133936072602491412737245870066063155881748815209209628292540917153643678925903600113305305488204665213841469519415116094330572703657595919530921861173819326117931051185480744623799627495673518857527248912279381830119491298336733624406566430860213949463952247371907021798609437027705392171762931767523846748184676694051320005681271452635608277857713427577896091736371787214684409012249534301465495853710507922796892589235420199561121290219608640344181598136297747713099605187072113499999983729780499510597317328160963185950244594553469083026425223082533446850352619311881710100031378387528865875332083814206171776691473035982534904287554687311595628638823537875937519577818577805321712268066130019278766111959092164201989380952572010654858632788659361533818279682303019520353018529689957736225994138912497217752834791315155748572424541506959508295331168617278558890750983817546374649393192550604009277016711390098488240128583616035637076601047101819429555961989467678374494482553797747268471040475346462080466842590694912933136770289891521047521620569660240580381501935112533824300355876402474964732639141992726042699227967823547816360093417216412199245863150302861829745557067498385054945885869269956909272107975093029553211653449872027559602364806654991198818347977535663698074265425278625518184175746728909777727938000816470600161452491921732172147723501414419735685481613611573525521334757418494684385233239073941433345477624168625189835694855620992192221842725502542568876717904946016534668049886272327917860857843838279679766814541009538837863609506800642251252051173929848960841284886269456042419652850222106611863067442786220391949450471237137869609563643719172874677646575739624138908658326459958133904780275901
```

### 잡에서 여러 파드 실행하기

- 비 병렬 작업:
일반적으로 Pod가 실패하지 않는 한 Pod는 하나만 시작됩니다.
Pod가 성공적으로 종료되면 Job은 완료됩니다.

- 고정 완료 횟수를 가진 병렬 작업:
.spec.completions에 0 이상의 양수 값을 지정합니다.
Job은 전체 작업을 나타내며, .spec.completions 개의 성공한 Pod가 있을 때 완료됩니다.
.spec.completionMode="Indexed"를 사용할 때, 각 Pod는 0부터 .spec.completions-1까지 다른 인덱스를 가집니다.

- 작업 큐를 사용하는 병렬 작업:
.spec.completions을 지정하지 않으면 기본적으로 .spec.parallelism을 사용합니다.
Pod는 자체적으로 또는 외부 서비스를 통해 각자 작업할 내용을 결정하기 위해 협력해야 합니다. 예를 들어, Pod는 작업 큐에서 최대 N개의 항목을 가져올 수 있습니다.
각 Pod는 독립적으로 모든 동료 Pod가 완료되었는지 여부를 판단하고, 따라서 전체 Job이 완료되었는지 여부를 알 수 있습니다.
Job에서 하나의 Pod가 성공적으로 종료되면 새로운 Pod는 생성되지 않습니다.
최소한 하나의 Pod가 성공적으로 종료되었고 모든 Pod가 종료된 후에 Job은 성공으로 완료됩니다.
하나의 Pod가 성공적으로 종료되었을 때, 다른 Pod는 해당 작업 또는 출력에 대해 더 이상 작업을 수행하지 않아야 합니다. 모든 Pod는 종료 프로세스에 참여해야 합니다.
위의 영어 설명에 따라서 잡은 2개 이상의 파드 인스턴스를 생성해 병렬 또는 순차적으로 실행하도록 구성할 수 있다.


```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  completions: 5 
  template:
    spec:
      containers:
        - name: pi
          image: perl:5.34.0
          command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4

```

completions를 5로 설정하면 이 잡은 다섯개의 파드를 순차적으로 실행한다.


```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  completions: 5 
  parallelism: 2
  template:
    spec:
      containers:
        - name: pi
          image: perl:5.34.0
          command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
```

잡 파드는 하나씩 차례로 실행하는 대신 잡이 여러 파드를 병렬로 실행할 수도 있다.

> [Job](https://kubernetes.io/ko/docs/concepts/workloads/controllers/job/)  

### 잡스케일링

잡이 실행되는 동안 parallelism 속성을 변경할 수도 있다. 이것은 레플리카셋이나 레플리케이션 컨트롤러를 스케일링 하는 것과 유사하며, kubectl scale 명령을
사용해 수행할 수 있다.

```shell
k scale job multi-completion-batch-job --replicas 3 
```

### 잡 파드가 완료되는데 걸리는 시간 제한하기

파드 스펙에 activeDeadlineSeconds 속성을 설정해 파드의 실행 시간을 제한할 수 있다. 파드가 이보다 오래 실행되면 시스템이 종료를 시도하고
잡을 실패한 것으로 표시한다.

### 잡을 주기적으로 또는 한 번 실행되도록 스케쥴링하기

쿠버네티스에서의 크론작업은 크론잡 리소스를 만들어 구성한다. 잡 실행을 위한 스케쥴은 잘 알려진 크론 형식으로 지정하므로, 일반적인 크론 작업에 익숙하다면
금방 쿠버네티스의 크론 잡을 이해할 수 있을 것이다.


```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: hello
              image: busybox:1.28
              imagePullPolicy: IfNotPresent
              command:
                - /bin/sh
                - -c
                - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

스케쥴 설정하기
- 분
- 시
- 일
- 월
- 요일

예시) 0,15,30,45 * * * * : 이는 매시간, 매일, 매월, 모든 요일의 0, 15, 30, 45 분에 실행됨을 의미한다.

```shell
k create -f ~/sources/02_linesgits/lines_kubernetes/007_kuberntes_in_action/p200_cron_jobs/cron_jobs_sample.yaml 

k get cronjob

k delete cronjob.batch/hello
```

## References 

> [How to find all ReplicaSets that have not enough running replicas](https://github.com/kubernetes/kubectl/issues/717)    
>   - [From a string, formatted as a table with spaces, how to grep all values that are equal for all columns?](https://serverfault.com/questions/985559/from-a-string-formatted-as-a-table-with-spaces-how-to-grep-all-values-that-are/985578#985578)      
> [ReplicaSet](https://subicura.com/k8s/guide/replicaset.html#replicaset-%E1%84%86%E1%85%A1%E1%86%AB%E1%84%83%E1%85%B3%E1%86%AF%E1%84%80%E1%85%B5)     
> [중단(disruption)](https://kubernetes.io/ko/docs/concepts/workloads/pods/disruptions/)   