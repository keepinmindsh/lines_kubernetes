# Section 13 - 스테이트풀셋에 대하여 

## 스테이트풀셋 vs 레플리카셋

- 레플리카셋 (스테이스리스)

기존의 인스턴스가 죽더라도 새로운 인스턴스를 만들 수 있고,
기존의 인스턴스가 가지고 있던 아이덴티티에 대해서 고민할 필요가 없다.

- 스테이트풀 애플리케이션

애플리케이션의 경우 새 인스턴스가 이전 인스턴스와 완전희 같은 상태와 아이덴티티를 가져야함을 의미 한다.

**즉 각각의 앱이 다른 역할을 가지는 용도로 사용될 때**

스테이트풀 파드는 종료되면 새로운 파드 인스턴스는 교체되는 파드와 동일한 이름 , 네트워크 아이텐티디, 상태 그대로 다른 노드에서 되살아나야 한다.

각 새로운 파드 인스턴스가 완전히 무작위가 아닌 에측 가능한 아이덴티티를 가진다.

- 안정적인 네트워크 아이덴티티 제공하기

스테이트풀 셋으로 생성된 파드는 서수 인덱스가 할당되고 파드의 이름과 호스트 이름, 안정적인 스토리지를 붙이는데 사용된다. 스테이트 풀렛의 이름과 인스턴스의 서수 인덱스로부터 파생되므로 파드의 이름을 예측할 수 있다.  
**파드의 이름의 이름이 아닌 잘 정리된 이름을 갖는다.**

- 거버닝 서비스 소개

스테이트풀 파드는 각각 서로 다르므로 그룹의 특정 파드에서 동작하기를 원할 것이다.

**스테이트풀셋**은 거버닝 헤드리스 서비스를 생성해서 각 파드에게 실제 네트워크 아이덴티티를 제공해야 한다.
이 서비스를 통해 각 파드는 자체 DNS 엔트리를 가지며 클러스터의 피어 혹은 클러스터의 다른 클라이언트가 호스트
이름을 통해 파드의 주소를 지정할 수 있다.


예시)  
default라는 네임스페이스에 속하는 foo라는 이름의 거버닝 서비스가 있고 파드의 이름이 A-0이라면, 이 파드는 a-o.foo.default.svc.cluster.local 이라는
FQDN을 통해 접근할 수 있다. 레플리카 셋으로 관리되는 파드에서는 불가능하다.  
또한 foo.default.svc.cluster.local 도메인의 SRV 레코드를 조회해 모든 스테이트 풀셋의 파드이름을 찾는 목적으로 DNS를 사용할 수 있다.

- StatefulSet의 교체 방식

스테이트 풀셋으로 관리되는 파드 인스턴스가 사라지면 스테이트풀셋은 레플리카셋이 하는 것과 비슷하게 새로운 인스턴스로 교체되도록 한다. 하지만 레플리카셋과 달리 교체된 파드는 사라진 파드와 동일한 이름과 호스트 이름을 갖는다.

- StatefulSet의 스케일링

스테이트풀셋을 스케일링하면 사용하지 않는 다음 서수 인덱스를 갖는 새로운 파드 인스턴스를 생성한다.
인스턴스 두 개에서 세 개로 스케일업하면 새로운 인스턴스는 인덱스 2를 부여받는다.
스테이트 풀셋의 스케일 다운의 좋은 점은 항상 어떤 파드가 제거될 지 알수 있다는 점이다. 다시 말하자면
이는 어떤 인스턴스가 삭제될지 알수 없고, 어떤 인스턴스를 먼저 제거할지 지정할 수 없는 레플리카 셋의 스케일 다운과
대조적이다. 스테이트 풀셋의 스케일 다운은 항상 가장 높은 서수 인덱스를 먼저 제거한다. 따라서 스케일 다운의 영향을 예측할 수 있다.

- Storage에 대한 StatefulSet의 적용

각 스테이트풀 파드 인스턴스는 자체 스토리지를 사용할 필요가 있고 스테이트풀 파드가 다시 스케쥴링되면,
새로운 인스턴스는 동일한 스토리지에 연결돼야 한다. 스테이트풀 파드의 스토리지는 영구적이어야 하고 파드와
분리돼야 한다.

이를 위해서 우리는 퍼시스턴트 볼륨 클레임과 퍼시스턴트 볼륨을 이용해서 검토해보겠다.

퍼시스턴트 볼륨 클레임이 퍼시스턴트 볼륨에 일대일로 매핑돼때, 스테이트풀 셋이 각 파드는 별도의 퍼시스턴트 볼륨을 갖는
다른 퍼시스턴트 볼륨 클레임을 참조해야 한다.

- 볼름 클레임 템플릿과 파드 템플릿을 동시 구성

스테이트 풀셋이 파드를 생성하는 것과 같은 방식으로 퍼시스턴트 볼륨 클레임 또한 생성해야 한다. 이런 이유로 스테이트풀렛은
각 파드와 함께하는 퍼시스턴트 볼륨 클레임을 복제하는 하나 이상의 불륨 클레임 템플릿을 가질 수 있다.

- 퍼시스턴트볼륨 클레임의 생성과 삭제의 이해

스테이트풀셋을 하나의 스케일 업하면 두 개 이상의 API 오브젝트가 생성된다. 하지만 스케일 다운을 할 대 파드만 삭제하고
클레임은 남겨둔다. 클레임이 삭제된 경우 어떤 일이 생길지 생각해보면 그 이유는 분명하다. 클레임이 삭제된 후 바인딩 됐던
퍼시스턴트 볼륨은 재활용되거나 삭제돼 콘텐츠가 손실된다.

따라서 스테이트풀 파드는 스테이트 풀 애플리케이션을 실행하기 위한 것으로 불륨에 저장하는 데이터가 중요하고 스테이트 풀셋의 스케일 다운에서 클레임이 삭제되면 결과는 치명적인 문제가 된다.

이런 이유로 기반 퍼시스턴트 볼륨을 해제하려면 퍼시스턴트 볼륨 클레임을 수동으로 삭제해야 한다.

- 동일 파드의 새 인스턴스에 퍼시스턴트볼륨클레임 다시 붙이기

스케일 다운 이후 퍼시스턴트볼륨 클레임이 남아 있다는 사실은 이후에 스케일업하면 퍼시스턴트 볼륨에 바인딩된 동일한 클레임을 연결할수 있고 새로운 파드에 그 콘텐츠가 연결된다는 것을 의미한다.

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/statefulset_001.png)

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/statefulset_002.png)

## 스테이트풀셋 사용하기

- StatefulSet의 기본적인 사용 방식

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # by default is 1
  minReadySeconds: 10 # by default is 0
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: registry.k8s.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "my-storage-class"
      resources:
        requests:
          storage: 1Gi
```

> [https://kubernetes.io/docs/concepts/storage/persistent-volumes/](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)  
> [https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)

- Persistence Volume을 연동하는 StatefulSet


```shell
$ gcloud compute disks create --size=lGiB --zone=europe -westl-b pv-a
$ gcloud compute disks create --size=lGiB --zone=europe -westl-b pv-b
$ gcloud compute disks create --size=lGiB --zone=europe -westl-b pv-c
```

```yaml 
kind: List 
apiVersion: v1 
items: 
- apiVersion: v1 
  kind: PersistenceVolume 
  metadata: 
    name: pv-a 
  spec: 
    capacity:
      storage: 1Mi 
    accessModes: 
      - ReadWriteOnce
    persistentVolumeReclaimPolicy: Recycle 
    gcePersistentDisk: 
      pdName: pv-a 
      fsType: nfs4
```


```yaml
apiVersion: v1 
kind: Service 
metadata: 
  name: kubia 
spec: 
  clusterIP: None 
selector: 
  app: kubia 
ports: 
  - name: http 
    port: 80 
```

```yaml
apiVersion: apps/v1beta1 
kind: StatefulSet 
metadata: 
  name: kubia 
spec: 
  serviceName: kubia 
  replicas: 2 
  template: 
    metadata: 
      labels: 
        app: kubia 
    spec: 
      containers: 
      - name: kubia 
        image: luksa/kubia-pet
        ports: 
        - name: http 
          containerPort: 8080
        volumeMounts: 
        - name: data 
          mountPath: /var/data 
    volumeClaimTemplates:
    - metadata: 
        name: data 
      spec: 
        accessModes:
        - ReadWriteOnce
        resources: 
          requests: 
          storage: 1Mi
```

```shell
$ kubectl create -f kubia-statefulset.yaml
```

스테이트풀셋의 경우 첫 번째 파드가 생성되고 준비가 완료돼야 두 번째 파드가 생성된다. 특정 클러스터된 스테이트풀 애플리케이션은 두 개 이상의 멤버가 동시에
생성되면 레이스 컨디션에 빠질 가능성이 있기 때문에 스테이트풀셋은 이와 같이 동작한다. 나머지 멤버를 계속 기동하기 전에 각 멤버가 완전히 기동되게 하는 것이 안전하다.

```shell
# 생성된 스테이트 풀 셋으로 생성된 스테이트풀 파드 
$ kubectl get po kubia-0 -o yaml 
```

```shell
# 생성된 퍼시스턴트 볼륨 클레임 살펴보기 
$ kubectl get pvc 
```

- statefulset 스케일링

스테이트풀셋의 스케일 다운은 파드를 삭제하지만 퍼시스턴트 볼륨 클레임은 변경되지 않은 상태로 유지된다.   
또한 스케일 다운은 점진적으로 수행되며 스테이트풀셋이 초기에 생성됐을 때 개별 파드가 생성되는 방식과 유사하다. 하나 이상의  
인스턴스를 스케일 다운 하면 가장 높은 서수의 파드가 먼저 삭제된다.

## 스테이트풀셋의 피어 디스커버리


## 스테이트풀셋이 노드 실패를 처리하는 과정 이해하기 