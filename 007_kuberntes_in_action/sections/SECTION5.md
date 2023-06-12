# Section 5 - 레플리카셋

## 레플리카셋

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
apiVersion: apps/v1beta2
kind: ReplicaSet
metadata:
  name: kubia
spec:
  replicas: 3
  selector:
    matchLabels:
      app: kubia
  template:
    metadata:
      labels:
        app: kubia
    spec:
      containers:
      - name: kubia
        image: luksa/kubia 
```

- selector ~ : 여기서는 레플리케이션컨트롤러와 유사한 간단한 matchLabels 셀렉터를 사용한다.
- template ~ : 템플릿은 레플리케이션 컨트롤러와 동일하다.

> API 버전 속성에 대해서
> - API 그룹 ( apps )
> - API 버전 ( v1beta2 )
    > core API 그룹에 속하는 어떤 쿠버네티스 리소스들은 apiVersion 필드를 지정할 필요가 없다는 것을 책 전체에 걸쳐서 보게 될 것이다.
    > 최신 쿠버네티스 버전에서 도입된 다른 리소스는 여러 API 그룹으로 분류된다.

### 리플리카셋 생성, 검사

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
- DoesNotExist는 파다에 지정된 키를 가진 레이블이 포함돼 있지 않아야 한다. 값 필드를 지정하지 않아야 한다.

### 레플리카셋 정리

```yaml
$ k delete rs kubia 
```

## 데몬셋

### 데몬셋을 사용해 각 노드에서 정확히 한개의 파드 실행

클러스터의 모든 노드에, 노드당 하나의 파드가 실행되길 원하는 경우가 있을 수 있다.

예) 로그수집기, 리소스 모니터, Kube-proxy 프로세스 등등

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

### 필요한 레이블을 노드에 추가하기

```shell
k get node 

k lebel node {node name} disk=ssd

k get po   
```

데몬셋을 삭제하거나 대상으로 타케팅된 레이블이 변경될 경우 daemonSet에 의해서 생성된 파드도 삭제된다.

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

- Non-parallel Jobs
  normally, only one Pod is started, unless the Pod fails.
  the Job is complete as soon as its Pod terminates successfully.
- Parallel Jobs with a fixed completion count:
  specify a non-zero positive value for .spec.completions.
  the Job represents the overall task, and is complete when there are .spec.completions successful Pods.
  when using .spec.completionMode="Indexed", each Pod gets a different index in the range 0 to .spec.completions-1.
- Parallel Jobs with a work queue:
  do not specify .spec.completions, default to .spec.parallelism.
  the Pods must coordinate amongst themselves or an external service to determine what each should work on. For example, a Pod might fetch a batch of up to N items from the work queue.
  each Pod is independently capable of determining whether or not all its peers are done, and thus that the entire Job is done.
  when any Pod from the Job terminates with success, no new Pods are created.
  once at least one Pod has terminated with success and all Pods are terminated, then the Job is completed with success.
  once any Pod has exited with success, no other Pod should still be doing any work for this task or writing any output. They should all be in the process of exiting.

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