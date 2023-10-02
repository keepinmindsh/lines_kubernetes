# Section 12 - 배포에 대하여 

## Cheat Sheet 

```shell
$ kubectl create deployment my-dep --image=busybox
```

```shell
$ kubectl create deployment my-dep --image=busybox -- date
```

```shell
$ kubectl create deployment my-dep --image=nginx --replicas=3
```

```shell
$ kubectl create deployment my-dep --image=busybox --port=5701
```

```shell
$ kubectl rollout undo deployment/abc 

$ kubectl rollout status daemonset/foo

$ kubectl rollout history deployment/abc

$ kubectl rollout history daemonset/abc --revision=3

$ kubectl rollout pause deployment/nginx

$ kubectl rollout restart deployment/nginx 

$ kubectl rollout restart daemonset/abc

$ kubectl rollout restart deployments/abc -n {name_space} 

$ kubectl rollout resume deployment/nginx

$ kubectl rollout status deployment/nginx

$ kubectl rollout undo deployment/abc

$ kubectl rollout undo daemonset/abc --to-revision=3 

$ kubectl rollout undo --dry-run=server deployment/abc
```

## 파드에서 실행 중인 애플리케이션 업데이트

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/pod_application_update.png)

처음에는 파드가 애플리케이션의 첫 번째 버전을 실행한다. 이미지에 v1 태그가 지정돼 있다고 가정해보자. 그런 다음 최신 버전의 애플리케이션을 개발해
v2로 태그가 지정된 새 이미지를 이미지 저장소에 푸시한다. 다음으로 모든 파드를 이 새 버전으로 바꾸려고 한다. 파드를 만든 후에는 기존 파드의 이미지를
변경할 수 없으므로 기존 파드를 제거하고 새 이미지를 실행하는 새 파드로 교체해야 한다.

파드를 만든 후에는 기존 파드의 이미지를 변경할 수 없으므모 기존 파드를 제거하고 새 이미지를 실행하는 새 파드로 교체헤야 한다.

파드를 교체하는 2가지 전략이 있다.

- 기존 파드를 모두 삭제한 다음 새 파드를 시작한다.
- 새로운 파드를 시작하고, 기동하면 기존 파드를 삭제한다. 새 파드를 모두 추가한다음 한꺼번에 기존 파드를 삭제하거나 순차적으로 새파드를 추가하고 기존 파드를 점진적으로 제거해 이 작업을 수행할 수 있다.

이 두 가지 전략 모두 장단점이 있다. 첫 번째는 짧은 시간 동안 애플리케이션을 사용할 수 없다. 두 번째를 사용하면 애플리케이션이 동시에 두 가지 버전을 실행해야 한다.  
애플리케이션이 데이터 저장소에 데이터를 저장하는 경우 새 버전이 이전 버전을 손상 시킬 수 있는 데이터 스키마나 데이터의 수정을 해서는 안된다.

### 오래된 파드를 삭제하고 새 파드로 교체

모든 파드 인스턴스를 새 버전의 파드로 교체하기 위해 레플리케이션 컨트롤러를 사용하는 방법을 알고 있을 것이다.   
레플리케이션컨트롤러의 파드 템플릿은 언제든지 업데이트 할 수 있다. 레플리케이션 컨트롤러는 새 인스턴스를 생성할 때 업데이트된 파드 템플릿을 사용한다.  
레플리케이션 컨트롤러는 레이블 셀렉터와 일치하는 파드가 없다면 새 인스턴스를 시작한다.

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/replication_controller_pod_replace.png)

### 새 파드 기동과 이전 파드 삭제

다운 타임이 발생하지 않고 한 번에 여러 버전의 애플리케이션이 실행하는 것을 지원하는 경우 프로세스를 먼저 전환해 새 파드를 모두 기동한후
이전 파드를 삭제할 수 있다. 잠시동안 동시에 두 배의 파드가 실행되므로 하드웨어 리소스가 필요하다.

- 한 번에 이전 버전에서 새 버전으로 전환

파드의 앞쪽에서는 일반적으로 서비스를 배치한다. 새 버전을 실행하는 파드를 불러오는 동안 서비스는 파드의 이전 버전에 연결된다.   
서비스의 레이블 셀렉터를 변경하고 서비스를 새 파드로 전환할 수 있다. 이것을 블루-그린 디플로이 먼트라고 한다.  
전환한 후 새 버전이 올바르게 작동하면 이전 레플리케이션 컨트롤러를 삭제해 이전 파드를 삭제할 수 있다.

> kubectl set selector 명령어를 사용해 서비스의 파드 셀렉터를 변경할 수 있다.

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/service_replacement.png)

- 롤링 업데이트 수행

새 파드가 모두 실행된 후 이전 파드를 한 번에 삭제하는 방법 대신 파드를 단계별로 교체하는 롤링 업데이트를 수행할 수도 있다.  이전 레플리케이션 컨트롤러를 천천히  
스케일 다운하고 새 파드를 스케일 업해 이를 수행할 수 있다. 이 경우 서비스의 파드 셀렉터에 이전 파드와 새 파드를 모두 포함하게 해 요 청을 두 파드 세트로 보낼 수 있다.

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/rolling_update.png)

> p394


## 레플리케이션 컨트롤러 자동 롤링 업데이트 수행

-  v1 버전의 애플리케이션 생성

```javascript
const http = require('http')
const os = require('os')
console.log("Kubia server starting....")
var handler = function(request, response) {
    console.log("Received request from " + request.connection.remoteAddress);
    response.writeHead(200);
    response.end("This is v1 running in pod " + os.hostname() + "\n");
}
var www = http.createServer(handler);
www.listen(8080);
```

```yaml
apiVersion: v1 
kind: ReplicationController 
metadata: 
  name: kubia-v1 
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
--- 
apiVersion: v1 
kind: Service 
metadata: 
  name: kubia 
spec: 
  type: LoadBalancer 
  selector: 
    app: kubia 
  ports: 
  - port: 80 
    targetPort: 8080 
```

> 현재는 Replication Controller는 ReplicaSet에 의해서 대체 되었는데,
> ReplicaSet의 차이점은 복제할 수를 관리하기 위한 Selector를 필수적으로 정의해야 한다.

```yaml
apiVersion: apps/v1 
kind: ReplicaSet
metadata: 
  name: nginx 
spec: 
  replicas: 3 
  selector: 
    matchExpressions:
      { key: app, operator: In, values: [nginx, frontend]}
      {key : environment, operator: NotIn, values: [production] }
  template: 
    metadata: 
      labels:
        app: nginx 
        environment: dev 
    spec: 
      containers: 
        name: nginx 
        image: nginx 
        ports: 
          containerPort: 80
```

### kubectl을 이용한 롤링 업데이트

```shell
$ kubectl rolling-update kubia-v1 kubia-v2 --image=luksa/kubia:v2 
```

레플리케이션 컨트롤러 kubia-v1 을 실행중인 하나의 kubia 애플리케이션을 버전 2로 교체했기 때문에 새로운 레플리케이션 컨트롤러를 kubia-v2라고 하고 luksa/kubia:v2 컨트롤러 이미지를 사용한다.

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/rolling_update_02.png)


> **동일한 이미지 태그로 업데이트 푸시하기**   
> 애플리케이션을 수정하고 동일한 이미지 태그로 변경 사항을 푸시하는 것이 좋은 생각은 아니지만  
> 개발 중에는 그런 경향이 있다. 최신 태그를 수정하는 경우 문제가 되지 않지만 다른 태그로 이미지에  
> 태그를 다는 경우, 워커 노드에서 일단 이미지를 한번 가져오면 이미지는 노드에 저장되고 동일한 이미지를  
> 사용해 새파드를 실행할 때 이미지를 다시 가져오지 않는다.   
> 즉, 변경한 내용을 같은 이미지 태그로 푸시하면 이미지가 변경되지 않는다. 새 파드가 동일한 노드로   
> 스케쥴된 경우 Kubelet은 이전 버전의 이미지를 실행한다. 반면 이전 버전을 실행하지 않은 노드는  
> 새 이미지를 가져와서 실행하므로 두 가지 다른 버전의 파드가 실행 될 수 있다. 이런 일이 발생하지 않도록  
> 하려면 컨테이트의 imagePullPolicy 속성을 Always로 설정해야 한다.

### 파드에 의한 Rolling Update의 문제

Replication Controller는 기존 버전의 파드 Scale Out을 모두 0으로 만들고 신규 버전으로 Scale Out을 3으로 변경하기 때문에
레플리케이션 컨트롤러가 프로덕션 클라이언트가 제공하는 모든 파드를 종료할 수도 있다!

위의 방식을 해결하기 위해서 레플리케이션 컨트롤러 두 개를 스케일업해 새 파드로 교체 한다.

이를 통해서 단계적으로 Pod 종료 및 재기동을 실행할 수 있다.

**다만,**

kubect를 통한 호출 방식은 쿠버네티스 API 서버로 보내는 각 HTTP 요청을 출력한다.

만약 kubectl이 업데이트를 수행하는 동안 네트워크 연결이 끊어진다면 어떨까?

업데이트 프로세스는 중간에 중단될 것이다. 파드와 레플리케이션 컨트로러는 중간 상태에서 끝이 난다.

해당 부분은 또한 선언형 방식이 아닌 실제 명령이 호출되는 것으로
선언형으로 정의하여 시스템의 상태를 내부적으로 스스로 찾아내어 달성할수 있는 구조가 되어야 한다.

직접 명령이 아니라 어떻게 변해야하는지만 전달하면 알아서 처리될 수 있는 구조라 만들어야 한다.


## 애플리케이션을 선언적으로 업데이트하기 위한 디플로이먼트 사용하기

디플로이먼트를 생성하면 레플리카셋 리소스가 그 아래에 생성된다. 레플리카셋은 차세대 래플리케이션 컨트롤러 이므로 레플리케이션 컨트롤러 대신 레플리카 셋을 사용해야 한다.

디플로이먼트를 사용하는 경우 실제 파드는 디플로이먼트가 아닌 디플로이 먼트의 레플리카셋에 의해 생성되고 관리된다.

### 디플로이먼트 

디플로이먼트는 파드와 레플리카셋에 대한 선언적 업데이트를 제공한다.  

디플로이먼트에서 의도하는 상태를 설명하고, 디플로이먼트 컨트롤러는 현재 상태에서 의도하는 상태로 비율을 조정하며 
변경한다. 새 레플리카셋을 생성하는 디플로이먼트를 정의하거나 기존 디플로이먼트를 제고하고, 모든 리소스를 새 디플로이먼트에  
적용할 수 있다. 

- 레플리카셋을 롤아웃 할 디플로이먼트 생성. 레플리카셋은 백그라운드에서 파드를 생성한다. 
- 디플로이먼트의 PodTemplateSpec을 업데이트 해서 파드의 새로운 상태를 선언한다.  
- 만약 디플로이먼트의 현재 상태가 안정적이지 않은 경우, 디플로이먼트의 이전 버전으로 롤백한다.  
- 더 많은 로드를 위하 디플로이먼트 스케일업 
- 디플로이먼트 롤아웃 일시 중지로 PodTemplateSpec에 여러 수정 사항을 적용하고, 재개하여 새로운 룰아웃을 시작함. 
- 롤아웃이 막혀있는지를 나타내는 디플로이먼트 상태를 이용 

### 디플로이먼트 생성

```yaml 
apiVersion: apps/v1beta1 
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
      - image: lukda/kubia:v1 
        name: nodejs
```

```shell 
$ kubectl delete rc --all 

$ kubectl create -f kubia-deployment-v1.yaml --record 

# 디플로이먼트 롤아웃 상태 출력 
$ kubectl rollout status deployment kubia 

$ kubectl get po

```

기본적으로 디플로이먼트는 선언을 하고 내부적으로 replicaset 오브젝트가 파드를 생성하는 역할을 수행한다.

### 디플로이먼트 업데이트

```shell 

$ kubectl set image deployment kubia nodejs=luksa/kubia:v2

# kubectl edit = 변경후 파일 저장하면 오브젝트 업데이트 
# kubectl patch = 오브젝트의 개별 속성을 수정한다. 
# kubectl apply = yaml/json 값을 수정해 오브젝트 수정한다. 
# kubectl replace = yaml/json 파일로 오브젝트를 새것으로 교체한다. 이명령은 오브젝트가 존재해야 한다. 
# kubectl set image = 컨테이너 이미지를 변경한다.  

```


### 디플로이먼트 롤백

```shell 
## 이전 버전으로 롤백 
$ kubectl rollout undo deployment kubia 

## 디플로이먼트 롤아웃 이력 표시 
$ kubectl rollout history deployment kubia 

## 특정 디플로이먼트 버전으로 롤백 
$ kubectl rollout undo deployment kubia --to-revision=1 

```

### 롤아웃 속도 제어

아래의 속성에 따라서 롤링 업데이트 중에 몇개의 파드를 교체할지를 결정한다.

```yaml 
spec: 
  staregy: 
    rollingUpdate: 
      macSurge: 1 
      maxUnavailable: 0
    type : RollingUpdate 
```

- maxSurge

디플로이먼트가 의도하는 레플리카수보다 얼마나 많은 파드 인스턴스 수를 허용할지를 결정한다. 기본적으로 25%로 설정되고 의도한 개수보다 최대 25% 더 많은 파드 인스턴스가 있을 수 있다.

- maxUnavailable

업데이트 중에 의도하는 레플리카 수를 기준으로 사용할 수 없는 파드 인스턴스 수를 결정한다. 또한 기본적으로 25%로 설정되고 사용 가능한 파드 인스턴스 수는 의도하는 레플리카 수의 75% 이하로 떨어지지 않아야 한다.

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/replicas_setting.png)

### 롤아웃 프로세스 일시 중지

```yaml
...
spec: 
  paused: false
...
```

```shell 

$ kubectl set image deployment kubia nodejs=luksa/kubia:v4

# 롤아웃 시작한 즉시 중시 
$ kubectl rollout pause deployment kubia 

```

새 파드 하나를 생성했지만 모든 원본 파드도 계속 실행중이어야 한다. 새 파드가 가동되면 서비스에 관한 모든 요청의 일부가 새파드로 전달된다.

이렇게 하면 카라니 릴리스를 효과적으로 실행할 수 있다.  
카나리 릴리스는 잘못된 버전의 애플리케이션이 롤아웃돼 모든 사용자에게 영향을 주는 위험을 최소화 하는 기술이다.

- 롤 아웃 재개

```shell 
$ kubectl rollout resume deployment kubia 
```

### 수정 버전 기록 제한 

롤백을 허용하기 위해 보존할 이전 레플리카셋의 수를 지정하는 선택적 필드이다. 이 이전 레플리카셋은 etcd의 리소스를 소비하고,  
kubectl get rs 의 결과를 가득차게 만든다. 각 디플로이먼트의 구성은 디플로이먼트의 레플리카셋에 저장된다. 이전 레플리카셋이 삭제되면  
해당 디플로이먼트의 수정 버전으로 롤백할 수 있는 기능이 사라진다.  

```yaml 
... 
spec: 
  revisionHistoryLimit: 
...
```

### 잘못된 버전의 롤아웃 방지

새 파드가 시작되자마자 레디니스 프로브가 매초마다 시작된다. 애플리케이션이 특정 요청 수부터 HTTP 상태 코드를 반환하기 때문에 특정 수 이후부터 레디니스 프로스가 실패하기 시작한다.

> minReadySeconds를 올바르게 설정하지 않고 레디니스 프로브만 정의하는 경우 레디니스
> 프로브의 첫 번째 호출이 성공하면 즉시 새 파드가 사용 가능한 것으로 간주된다. 레디니스 프로브가
> 곧 실패하면 모든 파드에서 잘못된 버전이 롤아웃된다. 따라서 minReadySeconds를 적절하게 설정해야 한다.

기본적으로 롤아웃이 10분 동안 진행되지 않으면 실패한 것으로 간주한다. 
