# Section 11 

## 파드 메타데이터와 그외의 리소스에 액세스 하기  

### Downward API 로 메타데이터 전달 

사용자가 데이터를 직접 설정하거나 파드가 노드에 스케줄링돼 실행되기 이전에 이미 알고 있는 데이터에는 적합하다. 그러나 파드의 IP, 호스트 노드 이름 또는 파드 자체의 
이름과 같이 실행 시점까지 알려지지 않은 데이터의 경우는 어떨까? 파드의 레이블이나 어노테이션과 같이 어딘가에 이미 설정된 데이터라면 어떨까?  
아마도 동일한 정보를 여러 곳에 반복해서 설정하고 싶지 않을 것이다.  

### 사용 가능한 메타데이터 이해 

아래의 정보를 컨테이너에 전달할 수 있다. 

- 파드의 이름 
- 파드의 IP 주소 
- 파드가 속한 네임스페이스 
- 파드가 실행중인 노드의 이름 
- 파드가 실행중인 서비스 어카운트 이름 
- 각 컨테이너의 CPU와 메모리 요청 
- 각 컨테이너의 CPU와 메모리 제한 
- 파드의 레이블 
- 파드의 어노테이션 

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/downward_flow.png)

#### 환경변수를 활용한 메타데이터 노출하기 

```yaml
apiVersion: v1 
kind: Pod 
metadata: 
  name: downward
spec: 
  containers: 
  - name: main
    image: busybox
    command: ["sleep","9999999"]
    resources: 
      requests: 
        cpu: 15m 
        memory: 100ki 
      limits: 
        cpu: 100m 
        memory: 4Mi 
    env: 
    - name: POD_NAME
      valueFrom: 
        fieldRef: 
          fieldPath: metadata.name 
    - name: POD_NAMESPACE  
      valueFrom: 
        fieldRef: 
          fieldPath: metadata.namespace 
    - name: POD_IP 
      valueFrom: 
        fieldRef: 
          fieldPath: metadata.namespace 
    - name: NODE_NAME 
      valueFrom: 
        fieldRef: 
          fieldPath: spec.nodeName
    ... 
```

프로세스가 실행되면 파드 스펙에 정의한 모든 환경 변수를 조회할 수 있다. 

```shell
$ kubectl exec downward env 
```

#### 파일로 메타데이터 전달 

```yaml
apiVersion: v1 
kind: Pod
metadata: 
  name: downward 
  labels: 
    foo: bar 
  annotations: 
    key1: value1 
    key2: |
      multi 
      line 
      value
spec: 
  containers: 
  - name: main  
    image: busybox 
    command: ["sleep", "9999999"]
    resources: 
      requests: 
        cpu: 15m 
        memory: 100Ki 
      limits:
        cpu: 100m 
        memory: 4Mi 
    volumeMounts: 
    - name: downward 
      mountPath: /etc/downward 
  volumes: 
  - name: downward 
    downwardAPI: 
      items: 
      - path: "podName"
        fieldRef: 
          fieldPath: metadata.name 
      - path: "podNamespace"
        fieldRef: 
          fieldPath: metadata.namespace 
      - path: "labels"
        fieldRef: 
          fieldPath: matadata.labels
      ...  
```

환경 변수로 메타데이터를 전달하는 대신 downward 라는 볼륨을 정의하고 컨테이너의 /etc/downward 아래에 마운트 한다.  
이 볼륨에 포함된 파일들은 볼륨 스펙의 downwardAPI.items 속성 아래에 설정된다.  

- 실제 내부 마운트된 정보를 확인해보면, 

```shell 
$ kubectl exec downward -- ls -1L /etc/downward  

$ kubectl exec downward cat /etc/downward/lables

$ kubectl exec downward cat /etc/downward/annotations
```

#### 레이블과 어노테이션 업데이트 

레이블이나 어노테이션이 변경될 때 쿠버네티스가 이 값을 가지고 있는 파일을 업데이트 해서 파드가 항상 최신 데이터를 볼 수 있도록 한다.  


> 컨피그 맵과 시크릿 볼륨과 마찬가지로 파드 스펙에서 downwardAPI 볼륨의 defaultMode 속성으로 파일 권한을 변경할 수 있다. 

#### 볼륨 스펙에서 컨테이너 수준의 메타데이터 참조 

```yaml
spec: 
  volumes: 
  - name: downward
    downwardAPI: 
      items: 
      - path: "containerCpuRequestMilliCores"
        resourceFieldRef: 
          containerName: main 
          resource: requests.cpu
          divisor: 1m
```

볼륨이 컨테이너가 아니라 파드 수준에서 정의되었다고 하면 그 이유가 분명해진다. 
볼륨 스펙 내에서 컨테이너의 리소스 필드를 참조할 때는 참조하는 컨테이너의 이름을 명시적으로 지정해야 한다. 
컨테이너가 하나인 파드에서 마찬가지다.  
볼륨을 사용해 컨테이너의 리소스 요청이나 제한을 노출하는 것은 환경변수를 사용하는 것보다 약간 더 복잡하지만 필요할 경우 
한 컨테이너의 리소스 필드를 다른 컨테이너에 전달할 수 있는 장점이 있다. 환경변수로는 컨테이너 자신의 리소스 제한과 
요청만 전달할 할 수 있다. 

## 쿠버네티스 API 서버와 통신하기 

다른 API 오브젝트에 관한 정보를 얻기 위해서 파드 내부에서 API 서버와 통신한다. 

```shell
$ kubectl cluster-info

$ kubectl proxy 
W0423 15:43:35.866535    3606 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
To learn more, consult https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke
Starting to serve on 127.0.0.1:8001

$ curl http://localhost:8001
{
  "paths": [
    "/.well-known/openid-configuration",
    "/api",
    "/api/v1",
    "/apis",
    "/apis/",
    "/apis/admissionregistration.k8s.io",
    "/apis/admissionregistration.k8s.io/v1",
    "/apis/apiextensions.k8s.io",
    "/apis/apiextensions.k8s.io/v1",
    "/apis/apiregistration.k8s.io",
    "/apis/apiregistration.k8s.io/v1",
    "/apis/apps",
    "/apis/apps/v1",
    "/apis/argoproj.io",
    "/apis/argoproj.io/v1alpha1",
    "/apis/authentication.k8s.io",
    "/apis/authentication.k8s.io/v1",
    "/apis/authorization.k8s.io",
    "/apis/authorization.k8s.io/v1",
    "/apis/auto.gke.io",
    "/apis/auto.gke.io/v1",
    "/apis/auto.gke.io/v1alpha1",
    "/apis/autoscaling",
    "/apis/autoscaling/v1",
    "/apis/autoscaling/v2",
    "/apis/autoscaling/v2beta1",
    "/apis/autoscaling/v2beta2",
    "/apis/batch",
    "/apis/batch/v1",
    "/apis/batch/v1beta1",
    "/apis/certificates.k8s.io",
    "/apis/certificates.k8s.io/v1",
    "/apis/cloud.google.com",
    "/apis/cloud.google.com/v1",
    "/apis/cloud.google.com/v1beta1",
    "/apis/coordination.k8s.io",
    "/apis/coordination.k8s.io/v1",
    "/apis/discovery.k8s.io",
    "/apis/discovery.k8s.io/v1",
    "/apis/discovery.k8s.io/v1beta1",
    "/apis/events.k8s.io",
    "/apis/events.k8s.io/v1",
    "/apis/flowcontrol.apiserver.k8s.io",
    "/apis/flowcontrol.apiserver.k8s.io/v1beta1",
    "/apis/flowcontrol.apiserver.k8s.io/v1beta2",
    "/apis/hub.gke.io",
    "/apis/hub.gke.io/v1",
    "/apis/internal.autoscaling.gke.io",
    "/apis/internal.autoscaling.gke.io/v1alpha1",
    "/apis/metrics.k8s.io",
    "/apis/metrics.k8s.io/v1beta1",
    "/apis/migration.k8s.io",
    "/apis/migration.k8s.io/v1alpha1",
    "/apis/networking.gke.io",
    "/apis/networking.gke.io/v1",
    "/apis/networking.gke.io/v1beta1",
    "/apis/networking.gke.io/v1beta2",
    "/apis/networking.k8s.io",
    "/apis/networking.k8s.io/v1",
    "/apis/node.k8s.io",
    "/apis/node.k8s.io/v1",
    "/apis/node.k8s.io/v1beta1",
    "/apis/nodemanagement.gke.io",
    "/apis/nodemanagement.gke.io/v1alpha1",
    "/apis/policy",
    "/apis/policy/v1",
    "/apis/policy/v1beta1",
    "/apis/rbac.authorization.k8s.io",
    "/apis/rbac.authorization.k8s.io/v1",
    "/apis/resolution.tekton.dev",
    "/apis/resolution.tekton.dev/v1alpha1",
    "/apis/resolution.tekton.dev/v1beta1",
    "/apis/scheduling.k8s.io",
    "/apis/scheduling.k8s.io/v1",
    "/apis/snapshot.storage.k8s.io",
    "/apis/snapshot.storage.k8s.io/v1",
    "/apis/snapshot.storage.k8s.io/v1beta1",
    "/apis/storage.k8s.io",
    "/apis/storage.k8s.io/v1",
    "/apis/storage.k8s.io/v1beta1",
    "/apis/tekton.dev",
    "/apis/tekton.dev/v1",
    "/apis/tekton.dev/v1alpha1",
    "/apis/tekton.dev/v1beta1",
    "/healthz",
    "/healthz/autoregister-completion",
    "/healthz/etcd",
    "/healthz/log",
    "/healthz/ping",
    "/healthz/poststarthook/aggregator-reload-proxy-client-cert",
    "/healthz/poststarthook/apiservice-openapi-controller",
    "/healthz/poststarthook/apiservice-openapiv3-controller",
    "/healthz/poststarthook/apiservice-registration-controller",
    "/healthz/poststarthook/apiservice-status-available-controller",
    "/healthz/poststarthook/bootstrap-controller",
    "/healthz/poststarthook/crd-informer-synced",
    "/healthz/poststarthook/generic-apiserver-start-informers",
    "/healthz/poststarthook/kube-apiserver-autoregistration",
    "/healthz/poststarthook/priority-and-fairness-config-consumer",
    "/healthz/poststarthook/priority-and-fairness-config-producer",
    "/healthz/poststarthook/priority-and-fairness-filter",
    "/healthz/poststarthook/rbac/bootstrap-roles",
    "/healthz/poststarthook/scheduling/bootstrap-system-priority-classes",
    "/healthz/poststarthook/start-apiextensions-controllers",
    "/healthz/poststarthook/start-apiextensions-informers",
    "/healthz/poststarthook/start-cluster-authentication-info-controller",
    "/healthz/poststarthook/start-kube-aggregator-informers",
    "/healthz/poststarthook/start-kube-apiserver-admission-initializer",
    "/livez",
    "/livez/autoregister-completion",
    "/livez/etcd",
    "/livez/log",
    "/livez/ping",
    "/livez/poststarthook/aggregator-reload-proxy-client-cert",
    "/livez/poststarthook/apiservice-openapi-controller",
    "/livez/poststarthook/apiservice-openapiv3-controller",
    "/livez/poststarthook/apiservice-registration-controller",
    "/livez/poststarthook/apiservice-status-available-controller",
    "/livez/poststarthook/bootstrap-controller",
    "/livez/poststarthook/crd-informer-synced",
    "/livez/poststarthook/generic-apiserver-start-informers",
    "/livez/poststarthook/kube-apiserver-autoregistration",
    "/livez/poststarthook/priority-and-fairness-config-consumer",
    "/livez/poststarthook/priority-and-fairness-config-producer",
    "/livez/poststarthook/priority-and-fairness-filter",
    "/livez/poststarthook/rbac/bootstrap-roles",
    "/livez/poststarthook/scheduling/bootstrap-system-priority-classes",
    "/livez/poststarthook/start-apiextensions-controllers",
    "/livez/poststarthook/start-apiextensions-informers",
    "/livez/poststarthook/start-cluster-authentication-info-controller",
    "/livez/poststarthook/start-kube-aggregator-informers",
    "/livez/poststarthook/start-kube-apiserver-admission-initializer",
    "/logs",
    "/metrics",
    "/openapi/v2",
    "/openapi/v3",
    "/openapi/v3/",
    "/openid/v1/jwks",
    "/readyz",
    "/readyz/autoregister-completion",
    "/readyz/etcd",
    "/readyz/informer-sync",
    "/readyz/log",
    "/readyz/ping",
    "/readyz/poststarthook/aggregator-reload-proxy-client-cert",
    "/readyz/poststarthook/apiservice-openapi-controller",
    "/readyz/poststarthook/apiservice-openapiv3-controller",
    "/readyz/poststarthook/apiservice-registration-controller",
    "/readyz/poststarthook/apiservice-status-available-controller",
    "/readyz/poststarthook/bootstrap-controller",
    "/readyz/poststarthook/crd-informer-synced",
    "/readyz/poststarthook/generic-apiserver-start-informers",
    "/readyz/poststarthook/kube-apiserver-autoregistration",
    "/readyz/poststarthook/priority-and-fairness-config-consumer",
    "/readyz/poststarthook/priority-and-fairness-config-producer",
    "/readyz/poststarthook/priority-and-fairness-filter",
    "/readyz/poststarthook/rbac/bootstrap-roles",
    "/readyz/poststarthook/scheduling/bootstrap-system-priority-classes",
    "/readyz/poststarthook/start-apiextensions-controllers",
    "/readyz/poststarthook/start-apiextensions-informers",
    "/readyz/poststarthook/start-cluster-authentication-info-controller",
    "/readyz/poststarthook/start-kube-aggregator-informers",
    "/readyz/poststarthook/start-kube-apiserver-admission-initializer",
    "/readyz/shutdown",
    "/version"
  ]
}%
```

### kubectl proxy로 쿠버네티스 API 살펴보기 

```shell
curl http://localhost:8001/apis/batch
{
  "kind": "APIGroup",
  "apiVersion": "v1",
  "name": "batch",
  "versions": [  ## 두가지 버전을 갖는 batch API 그룹 
    {
      "groupVersion": "batch/v1", 
      "version": "v1"
    },
    {
      "groupVersion": "batch/v1beta1",
      "version": "v1beta1"
    }
  ],
  "preferredVersion": {
    "groupVersion": "batch/v1",
    "version": "v1"
  }
}%      
```

- 사용 가능한 버전에 대해서 명시하여 조회하면 해당 버전에서 사용할 수 있는 resources를 보여준다. 

```shell
curl http://localhost:8001/apis/batch/v1 
{
  "kind": "APIResourceList",
  "apiVersion": "v1",
  "groupVersion": "batch/v1",
  "resources": [
    {
      "name": "cronjobs",
      "singularName": "",
      "namespaced": true,
      "kind": "CronJob",
      "verbs": [
        "create",
        "delete",
        "deletecollection",
        "get",
        "list",
        "patch",
        "update",
        "watch"
      ],
      "shortNames": [
        "cj"
      ],
      "categories": [
        "all"
      ],
      "storageVersionHash": "sd5LIXh4Fjs="
    },
    {
      "name": "cronjobs/status",
      "singularName": "",
      "namespaced": true,
      "kind": "CronJob",
      "verbs": [
        "get",
        "patch",
        "update"
      ]
    },
    {
      "name": "jobs",
      "singularName": "",
      "namespaced": true,
      "kind": "Job",
      "verbs": [
        "create",
        "delete",
        "deletecollection",
        "get",
        "list",
        "patch",
        "update",
        "watch"
      ],
      "categories": [
        "all"
      ],
      "storageVersionHash": "mudhfqk/qZY="
    },
    {
      "name": "jobs/status",
      "singularName": "",
      "namespaced": true,
      "kind": "Job",
      "verbs": [
        "get",
        "patch",
        "update"
      ]
    }
  ]
}% 
```

- 이후 실제 존재하는 Jobs를 조회하고 싶다면, 

아래의 결과는 현재 Kubernetes Object로 생성된 Jobs이 없기 때문에 Items에 배열이 빈값으로 표기된다.

```shell
curl http://localhost:8001/apis/batch/v1/jobs
{
  "kind": "JobList",
  "apiVersion": "batch/v1",
  "metadata": {
    "resourceVersion": "100856844"
  },
  "items": []
}% 
```

실제 Job을 생성후에 위의 명령어를 호출하면 Items에 Jobs가 포함되어 목록이 나오게 된다. 

- 이름별로 특정 값 인스턴스 검색하기 

명확하게 이름을 특정하여 아래와 같이 api 를 통해서 정의된 object 를 조회해올 수 있다. 

```shell
curl http://localhost:8001/apis/apps/v1/namespaces/default/deployments/lines-admin-nextjs-deployment
{
  "kind": "Deployment",
  "apiVersion": "apps/v1",
  "metadata": {
    "name": "lines-admin-nextjs-deployment",
    "namespace": "default",
    "uid": "a8dcd5bf-0aed-4175-8fd3-c68c456034a4",
    "resourceVersion": "94685604",
    "generation": 6,
    "creationTimestamp": "2023-01-12T13:11:58Z",
    "annotations": {
      "deployment.kubernetes.io/revision": "2",
....
```

위의 호출 방식과 똑같은 방식으로 아래와 같이 조회도 가능하다. 

```shell
$ k get deployment lines-admin-nextjs-deployment -o json
```

- [API 주소 찾기]

> 해당 부분 정상 동작되지 않음, 추후 재검토 필요! 

쿠버네티스 API 서버의 IP와 포트를 찾아야 한다. kubernetes라는 서비스가 디폴트 네임스페이스에 자동으로 노출되고 API 서버를 가리키도록 구성되기 때문에 쉽다. 

```shell
$ k get svc 


$ root@curl:/# env | grep KUBERNETES_SERVICE 

$ root@curl:/# curl https://kubernetes 
```

- 역할 기반 액세스 제어 비활성화 

RBAC가 활성화된 쿠버네티스 클러스터를 사용하는 경우 서비스 어카운트가 API 서버에 액세스할 권한이 없을 수 있다.  
API 서버를 쿼리할 수 있는 가장 간단한 방법은 다음 명령을 실행해 RBAC를 우회하는 것이다.   

```shell
$ k create clusterrolebinging permissive-binding \
          --clusterrole=cluster-admin \
          --group=system:serviceaccounts 
```

이렇게 하면 모든 서비스 어카운트에 클러스터 관리자 권한이 부여돼 원하는 방식돌 사용할 수 있다. 

- 서버의 아이덴티티 검증 

```shell
$ root@curl:/# ls /var/run/secrets/kubernetes.io/serviceaccout/

$ root@curl:/# curl --cacert /var/run/secrets/kubernets.io/serviceaccount

$ root@curl:/# export CURL_CA_BUNDLE=/var/run/secrets/kubernetes.io/ 

$ root@curl:/# TOKEN=$(cat /var/run/secrets/kubernetes.io)
``` 

- 파드가 실행 중인 네임스페이스 얻기

```shell
$ root@curl:/# NS=$(cat /var/run/secrets/kubernetes.io /serviceaccount/namespace)

$ root@curl:/# curl -H "Aurhorization: Bearer $TOKEN" https://kubernetes/api/v1/namespaces/$NS/pods 
```

- 파드가 쿠버네티스와 통신하는 방법 정리 

파드 내에서 실행 중인 애플리케이션이 쿠버네티스 API  에 적절히 액세스할 수 있는 방법을 정리해보자 

- 애플리케이션은 API 서버의 인증서가 인증 기관으로부터 서명됐는지를 검증해야하며, 인증 기관의 인증서는 ca.cert 파일에 있다. 
- 애플리케이션은 token 파일의 내용을 Authorization HTTP 헤더에 Bearer 토큰으로 넣어 전송해서 자신을 인증해야 한다. 
- namespace 파일은 파드의 네임스페이스 안에 있는 API 오브젝트의 CRUD 작업을 수행할 때 네임스페이스를 API 서버로 전달하는 데 사용해야 한다. 

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/default-token.png)

### 앰배서더 컨테이너를 이용한 API 서버 통신 간소화  

HTTPS, 인증서, 인증 토큰을 다루는 일은 때때로 개발자에게 너무 복잡할 때가 있다. 
보안을 유지하면서 통신을 훨씬 가단하게 만들 수 있는 방법이 있다. 

- 앰배서더 컨테이너 소개  

API 서버를 쿼리해야 하는 애플리케이션이 있다고 상상해보자. API 서버와 직접 통신하는 대신 메인 컨테이너 옆의 앰배서더 컨테이너에서 kubectl proxy를  
실행하고 이를 통해 API 서버와 통신할 수 있다.  
API 서버와 직접 통신하는 대신 메인 컨테이너의 애플리케이션은 HTTPS 대신 HTTP로 앰배서더에 연결하고 앰배서더 프록시가 API 서버에 대한 HTTPS 연결을  
처리하도록 해 보안을 투명하게 관리할 수 있다. 시크릿 볼륨에 있는 default-token v파일을 사용해 이를 수행한다. 

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/ambassador_container.png)

파드의 모든 컨테이너는 동일한 루프백 네트워크 인터페이스를 공유하므로 애플리케이션은 localhost 의 포트로 프록시에 액세스할 수 있다. 

- 추가적인 앰배서더 컨테이너를 사용한 curl 파드 실행 

```yaml
apiVersion: v1 
kind: Pod 
metadata: 
  name: curl-with-ambassador 
spec: 
  containers: 
  - name: main 
    image: tutum/curl 
    command: ["sleep", "9999999"]
  - name: ambassador 
    image: luksa/kubectl-proxy:1.6.2 
```

```shell
$ kubectl exec -it curl-with-ambassador -c main bash 
```

- ambassador를 통함 API 서버와의 통신 

> 파드 내부의 실행 권한이 정상적으로 동작하지 않아서 추후 검토 예정 

## 클라이언트 라이브러리를 사용해 API 서버와의 통신 

### 기본 요약 

- 파드의 이름, 네임스페이스 및 기타 메타데이터가 환경변수 또는 downward API 볼륨의 파일로 컨테이너 내부의 프로세스에 노출되는 방법 
- CPU와 메모리의 요청 및 제한이 필요한 단위로 애플리케이션에 전달되는 방법 
- 파드에서 downward API 볼륨을 사용해 파드가 살아있는 동안 변경 될 수 있는 최신 메타데이터 얻는 방법 
- kubectl proxy로 쿠버네티스 REST API를 탐색하는 방법 
- 쿠버네티스에 정의된 다른 서비스와 같은 방식으로 파드가 환경변수 또는 DNS 로 API 서버의 위치를 찾는 방법 
- 파드에서 실행되는 애플리케이션이 API 서버와 통신하는지 검증하고, 자신을 인증하는 방법 
- 클라이언트 라이브러리로 쉽게 쿠버네티스와 상호작용할 수 있는 방법 

# Section 12 

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

위의 방식을 해결학 위해서 레플리케이션 컨트롤러 두 개를 스케일업해 새 파드로 교체 한다.   

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
$ kubectl rollout und deployment kubia --to-revision=1 

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

디플로이먼트가 의도하는 레플라키수보다 얼마나 많은 파드 인스턴스 수를 허용할지를 결정한다. 기본적으로 25%로 설정되고 의도한 개수보다 최대 25% 더 많은 파드 인스턴스가 있을 수 있다. 

- maxUnavailable 

업데이트 중에 의도하는 레플리카 수를 기준으로 사용할 수 없는 파드 인스턴스 수를 결정한다. 또한 기본적으로 25%로 설정되고 사용 가능한 파드 인스턴스 수는 의도하는 레플리카 수의 75% 이하로 떨어지지 않아야 한다. 

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/replicas_setting.png)


### 롤아웃 프로세스 일시 중지 

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

### 잘못된 버전의 롤아웃 방지  

새 파드가 시작되자마자 레디니스 프로브가 매초마다 시작된다. 애플리케이션이 특정 요청 수부터 HTTP 상태 코드를 반환하기 때문에 특정 수 이후부터 레디니스 프로스가 실패하기 시작한다. 

> minReadySeconds를 올바르게 설정하지 않고 레디니스 프로브만 정의하는 경우 레디니스
> 프로브의 첫 번째 호출이 성공하면 즉시 새 파드가 사용 가능한 것으로 간주된다. 레디니스 프로브가
> 곧 실패하면 모든 파드에서 잘못된 버전이 롤아웃된다. 따라서 minReadySeconds를 적절하게 설정해야 한다.

기본적으로 롤아웃이 10분 동안 진행되지 않으면 실패한 것으로 간주한다. 

# Section 13

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

# Section 14 

## 아키텍처 이해 

- 쿠버네티스 컨트롤 플레인
  - etcd 분산 저장 스토리리 
  - API 서버 
  - 스케쥴러 
  - 컨트롤러 매니저 
- 워커노드 
  - Kubelet 
  - 쿠버네티스 서비스 프록시 ( kube-proxy )
  - 컨테이너 런타임 ( Docker, rkt 외 기타 )
- 애드온 구성 요소 
  - 쿠버네티스 DNS 서버 
  - 대시보드
  - 인그레스 컨트롤러 
  - 힙스터 
  - 컨테이너 네트워크 인터페이스 플러그인 

### 쿠버네티스 구성 요소의 분산 특성 

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_architecture_001.png)

쿠버네티스가 제공하는 모든 기능을 사용하려면, 이런 모든 구성 요소가 실행중이어야 한다. 

```shell
# 컨트롤 플레인 구성 요소의 상태 확인 
$ kubectl get componentstatuses
```

### 구성 요소가 서로 통신하는 방법 

쿠버네티스 시스템 구성요소는 오직 API 서버하고만 통신한다. 서로 직접 통신하지 않는다. API 서버는 etcd와 통신하는 유일한 구성요소다. 
kubectl을 이용해 로그를 가져오거나 kubectl attach 명령으로 실행 중인 컨테이너에 연결할 때 kubectl port-forward 명령을 실행할 대는 API
서버가 Kubelet에 접속한다. 

### 개별 구성 요소의 여러 인스턴스 실행 

워크 노드의 구성 여소는 모두 동일하 노드에서 실행돼야 하지만 컨트롤 플레인의 구성 요소는 여러서버에서 
실행될 수 있다. 

### 구성 요소 실행 방법 

kube-proxy와 같은 컨트롤 플레인 구성 요소는 시스템에 직접 배포하거나 파드로 실행할 수 있다.   
kubelet은 항상 일반 시스템 구성 요소로 실행되는 유일한 구성 요소이며, Kubelet이 다른 구성 요소를 파드로 실행한다. 
컨트롤 플레인 구성 요소를 파드로 실행하기 위해 Kubelet도 마스터 노드에 배포된다. 

```shell
$ kubectl get po -o custom-columns=POD:metadata.name,NODE:spec.nodeName --sort-by spec.nodeName -n kube-system
```

### 쿠버네티스가 etcd를 사용하는 방법 

모든 오브젝트는 API 서버가 다시 시작되거나 실패하더라도 유지하기 위해서 메니페스트가 영구적으로 저장될 필요가 있음. 
이를 위해서 쿠버네티스는 빠르고, 분산해서 저장되며, 일관된 키-값 저장소를 제공하는 etcd를 사용한다.  
쿠버네티스 API 서버 만이 etcd와 직접적으로 통신하는 유일한 구성요소다. 다른 구성 요소는 API로 간접적으로 데이터를 읽거나 
쓸수 있다. 

> 낙관적 동시성 제어에 관하여 검토 해볼 것! 

- 리소스를 etcd에 저장하는 방법 

```shell
$ etcdctl ls /registry

$ etcdctl ls /registry/pods 

$ etcdctl ls /registry/pods/default 

$ etcdctl ls /registry/pods/default/kubia-159041346-wt6ga 
```
> etcd API v3를 사용하는 경우 ls 명령을 사용해 디렉터리의 내용을 볼 수 없다. 
> 대신 etcdctl get /registry --prefix=true 명령을 이용해 지정한 접두사로 시작하는 모든 키를 표시할 수 없다. 

- 저장된 오브젝트의 일관성과 유효성 보장 

쿠버네티스는 다른 모든 구성 요소가 API 서버를 통하도록 함으로써 이를 개선했다. 
API 서버 한곳에서 낙관적 잠금 메커니즘을 구현해서 클러스터의 상태를 업데이트하기 때문에, 
오류가 발생할 가능성을 줄이고 항상 일관성을 가질 수 있다.

- 클러스터링된 etcd의 일관성 보장

etcd는 고가용성을 보장하기 위해서 두 개 이상의 etcd 인스턴스를 실행하는 것이 일반적이다. 
etcd는 RAFT 합의 알고리즘을 사용해 어느 순간이든 각 노드 상태가 대다수의 노드라 동의하는 현재 상태이거나 
이전에 동의된 상태 중에 하나임을 보장한다. 

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_architecture_002.png)

- etcd 인스턴스가 홀수 인 이유 

etcd는 인스턴스를 일반적으로 홀수로 배포한다. 두 개의 인스턴와 하나의 인스턴스를 비교해볼 때, 두개의 인스턴스가 있으며 
두 인스턴스 모두 과반이 필요하다. 둘중 하나라도 실패하면 과반이 존재하지 않기 때문에 상태를 변경할 수 없다. 두 개의 인스턴스를 갖는 것이 하나일 때 보다 오히러 더 좋지 않다. 
두개가 있으면, 전체 클러스터 장애 발생률이 단일 노드 클러스터에 비해 100% 증가된다. 세개와 네개에 대해서도 동일한다.  
 대규모 etcd 클러스터에서는 일반적으로 5대 혹은 7대 노드면 충분하다. 

### API 서버의 기능 

- 어드미션 컨트롤 플러그인 

리소스를 생성, 수정, 삭제하려는 요청인 경우에 해당 요청은 어드미션 컨트롤로 보내진다. 앞에서 말한것 처럼, 서버는 여러 어드미션 컨트롤 플러그인을 사용하도록 설정돼 있다.
이 플러그인은 리소스를 여러 가지 이유로 수정할 수 있다. 리소스 정의에 누락된 필드를 기본 값으로 초기화하거나 재정의할 수 있다. 

> [https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/)

### API 서버가 리소스 변경을 클라이언트에 통보하는 방법 이해 

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_architecture_003.png)

kubectl 도구는 리소스 변경을 감시 할 수 있는 API 서버의 클라이언트 중 하나다. 예를 들어 파드를 배포할 때 
kubectl get pods 명령을 반복 실행해 파드 리스트롤 조회할 필요가 없다. --watch 옵션을 이용해 파드의 생성, 수정, 삭제 통보를 받을 수 있다.  

```shell
$ kubectl get pods --watch 

$ kubectl get pods -o yaml --watch 
```

### 스케쥴러 이해

API 서버의 감시 메커니즘을 통해 새로 생성될 파드를 기다리고 있다가 할당된 노드가 없는 새로운 파드를 노드에 할당하기만 한다.   

스케줄러는 선택된 노드에 파드를 실행하도록 지시하지 않는다. 단지 스케쥴러는 API 서버는 Kubelet에 파드가 스케쥴링 된 것을 통보한다. 
대상 노드의 Kubelet은 파드가 해당 노드에 스케쥴링된 것을 확인하자마자 파드의 컨테이너를 생성하고 실행한다. 

- 기본 스케쥴링 알고리즘 이해 
  - 모든 노드 중에서 파드를 스케쥴링 할 수 있는 노드 목록을 필터링 한다. 
  - 수용 가능한 노드의 우선 순위를 정하고 점수가 높은 노드를 선택한다. 만약 여러 노드가 같은 최상위 점스를 가지고 있다면, 파드가 모든 노드에 고르게 배포되도록 라운드-로빈을 사용한다/ 

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_architecture_004.png)

- 수용 가능한 노드 찾기
  - 노드가 하드웨어 리소스에 대한 파드 요청을 충족시킬 수 있는가?
  - 노드에 리소스가 부족한가? 
  - 파드를 특정 노드로 스케쥴링하도록 요청한 경우에, 해당 노드인가? 
  - 노드가 파드 정의 안에 있는 노드 셀렉터와 일치하는 레이블을 가지고 있는가?
  - 파드가 특정 호스트 포트에 할당되도록 요청한 경우 해당 포트가 이 노드에서 이미 사용 중인가?
  - 파드 요청이 특정한 유형의 볼륨을 요청한 경우 이 노드에서 해당 볼륨을 파드에 마운트 할 수 있는가, 아니면 이 노드에 있는 다른 파드가 이미 같은 볼륨을 사용하고 있는가? 
  - 파드가 노드의 테인트를 허용하는가? 
  - 파드가 노드와 파드의 어피니티, 안티 어피니티 규칙을 지정했는가? 만약 그렇다면 이 노드에 파드를 스케쥴링 하면 이런 규칙을 어기게 되는가? 

- 파드에 가장 적합한 노드 선택 
- 고급 파드 스케줄링 
- 다중 스케줄러 사용 

### 컨트롤러 매니저에서 실행되는 컨트롤 소개 

- 레플리케이션 매니저
- 레플리카셋, 데몬섹, 잡 컨트롤러 
- 디플로이먼트 컨트롤러 
- 스테이트풀셋 컨트롤러 
- 노드 컨트롤러 
- 서비스 컨트롤러 
- 엔드포인트 컨트롤러 
- 네임스페이스 컨트롤러 
- 퍼시스턴트 볼륨 컨트롤러 
- 그 밖의 컨트롤러 

> [https://github.com/kubernetes/kubernetes/tree/master/pkg/controller](https://github.com/kubernetes/kubernetes/tree/master/pkg/controller)

### Kubelet이 하는 일 

Kubelet은 워커노드에서 실행하는 모든 것을 담당하는 구성 요소이다. 첫 번째 작업은 Kubelet이 실행 중인 
노드를 노드 리소스로 만들어서 API 서버에 등록하는 것이다. 그런 다음 API 서버를 지속적으로 모니터링해 해당 노드에 파드가 스케줄링되면, 
파드의 컨테이너를 시작한다. 설정된 컨테이너 런타임에 지정된 컨테이너 이미지로 컨테이너를 실행하도록 지시함으로써 이 작업을 수행한다. 
그런다은 Kubelet은 실행 중인 컨테이너를 계속 모니터링하면서 상태, 이벤트, 리소스 사용량을 API 서버에 보고한다.  

Kubelet은 컨ㅌ이너 라이브니스 프로브를 실행하는 구성 요소이기도 하며, 프로브가 실패할 경으 컨테이너를 다시 시작한다. 
마지막으로 API 서버에서 파드가 삭제되면 컨테이너를 정지하고 파드가 종료된 것을 서버에 통보한다. 

### 쿠버네티스 서비스 프록시의 역할 

Kubelet 외에도, 모든 워커 노드는 클라이언트가 쿠버네티스 API로 정의한 서빗에 연결할 수 있도록 해주는 kube-proxy 도 같이 실행한다. 
kube-proxy는 서비스의 IP와 포트로 들어온 접속을 서비스를 지원하는 파드 중 하나와 연결시켜준다.  
서비스가 둘 이상의 파드에서 지원되는 경우 프록시 파드 간에 로드 밸런싱을 수행한다. 

- 프록시라고 부르는 이유 

kube-proxy의 초기 구현은 사용자 공간에서 동작하는 프록시였다. 실제 서버 프로세스가 연결을 수락하고 이를 파드로 전달했다.  
서비스 IP로 향하는 연결을 가로채기 위해 프록시는 iptables 규칙을 설정해 이를 프록시 서버로 전송했다. 

kube-proxy는 실제 프록시이기 때문에 그 이름을 얻었지만, 현재는 훨씬 성능이 우수한 구현체에서 iptables 규칙만 사용해 프록시 서버를 
거치지 않고 패킷을 무작위로 선택한 백엔드 파드로 선택한다. 

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_architecture_005.png)

### 쿠버네티스 애드온 소개

- 애드온 배포 방식

```shell
$ kubectl get rc -n kube-system

$ kubectl get deploy -n kube-system  
```

- DNS 서버 동작 방식 
- 인그레스 컨트롤러 동작 방식 
- 다른 애드온 사용

## 컨트롤러가 협업하는 방법 

전체 프로세스를 시작하기 전에도 컨트롤러와 스케줄러 그리고 Kubelet은 API 서버에서 각 리소스 유형이 변경되는 것을 감시한다. 

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_architecture_006.png)

### 이벤트 체인 

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_architecture_007.png)

- 디플로이먼트가 레플리카셋 생성 
- 레플리카셋 컨트롤러가 파드 리소스 생성 
- 스케줄러가 생성한 파드에 노드 할당 
- Kubelet은 파드의 컨테이너를 실행한다. 

### 클러스터 이벤트 관찰 

kubectl describe 명령을 사용할 때 마다 특정 리소스와 관련된 이벤트를 봤지만, 
kubectl get events 명령을 이용해 이벤트를 직접 검색할 수도 있다. 

```shell
$ kubectl get events --watch 
```

## 실행중인 파드에 관한 이해

```shell
$ kubectl run nginx --image=nginx 
```

이제 ssh로 해당 파드가 실행 중인 워커 노드가 접속해 실행 중인 도커 컨테이너 목록을 살펴보자.  
GKE를 사용한다면 gcloud comput ssh 명령을 이용해 노드에 ssh로 접속할 수 있다.
노드에 들어가면, docker ps 명령으로 실행중인 컨테이너 목록을 나열할 수 있다. 

```shell
$ docker ps 
```
kubectl 을 통해서 pod 내의 컨테이너 생성시 pause 컨테이너는 파드의 모든 컨테이너가 동일한 네트워크와 리눅스 네임스페이스를 
공유하는 방법을 기억하는가? 퍼즈 컨테이너는 이러한 네임스페이스를 모두 보유하는 게 유일한 목적인 인프라스트럭처 컨테이너다. 
파드의 다른 사용자 정의 컨테이너는 파드 인프라 스트럭처 컨테이너의 네임스페이스를 사용한다.

## 파드간 네트워킹 

파드가 동일한 워커 노드에서 실행중인지 여부와 관계없이 파드끼리 서로 통신할 수 있어야 한다. 파드가 통신하는데 사용하는 네트워크는 파드가 보는 자신의 
IP 주소가 모든 다른 파드에서 해당 파드 주소를 찾을 때 정확히 동일한 IP 주소로 보이도록 해야 한다.  

파드 A가 네트워크 패킷을 보내기 위해 파드 B에 연결할 때, 파드 B가 보는 출발지 IP는 파드 A의 IP 주소와 동일해야 한다. 패킷은 네트워크 주소 변환  
없이 파드 A에서 파드 B로 출발지와 목적지 주소가 변경되지 않는 상태로 도착해야 한다. 

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_architecture_009.png)

이것은 매우 중요하다. 파드 내부에서 실행 중인 애플리케이션의 네트워킹이 동일한 네트워크 스위치에 접속한 시스템에서 실행되는 것처럼 간단하고 
정확하게 이뤄지도록 해주기 때문이다. 파드 사이에 NAT가 없으면 내부에서 실행중인 애플리케이션이 다른 파드에 자동으록 등록되도록 할 수 있다.  

### 네트워킹 동작 방식 자세히 알아보기

파드의 IP 주소와 네트워크 네임스페이스가 인프라스트럭처 컨테이너에 의해 설정되고 유지되는 것을 보았다. 파드의 컨테이너는 해당 네트워크 네임스페이스를 사용한다. 
따라서 파드의 네트워크 인터페이스는 인프라스트럭처 컨테이너에서 설정한것이다. 인터페이스를 생성하는 방법과 생성한 인터페이스를 모든 다른 파드 인터페이스에 연결하는 방법을 살펴보자. 

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_architecture_010.png)

- **동일한 노드에서 파드 간의 통신 활성화**

인프라스트럭처 컨테이너가 시작되기 전에, 컨테이너를 위한 가상 이더넷 인터페이스 쌍이 생성된다.  
이 쌍의 한쪽 인터페이스는 호스트의 네임스페이스에 남아 있고, 다른 쪽 인터페이스는 컨테이너의 네트워크 네임스페이스 안으로 옮겨져 이름이 eth0 으로 변경된다.  
두 개의 가상 인터페이스는 파이프의 양쪽 끝과 같다. 한쪽으로 들어가면 다른 쪽으로 나온다. 반대의 경우도 마찬가지다.  

호스트의 네트워크 네임스페이스에 있는 인터페이스는 컨테이너 런타임이 사용할 수 있도록 설정된 네트워크 브리지에 연결된다.  
컨테이너 안의 eth0 인터페이스는 브리지의 주소 범위안에서 IP를 할당받는다. 컨테이너 내부에서 실행되는 애플리케이션은 eth0 인터페이스로 전송하면,  
호스트 네임스페이스의 다른 쪽 veth 인터페이스로 나와 브리지로 전달된다. 이는 브리지에 연결된 모든 네트워크 인터페이스에서 수실할 수 있다는 것을 의미한다.  

파드 A에서 네트워크 패킷을 파드 B로 보내는 경우 먼저 패킨은 파드 A의 veth 쌍을 통해 프리지로 전달된 후 파드 B의 veth 쌍을 통과한다.  
노드에 있는 모든 컨테이너는 같은 브리지에 연결돼 있기 때문에 서로 통신할 수 있다.  
그러나 다른 노드에서 실행 중인 컨테이너가 서로 통신하려면 이 노드 사이의 브리지가 어떤 형태로든 연결돼야 한다.  

- **서로 다른 노드에서 파드 간의 통신 활성화**

서로 다른 노드 사이에 브리지를 연결하는 방법은 여러가지가 있다. 이는 오버레이, 언더레이 네트워크 아니면 일반적인 계층 3 라우팅을 통해 가능하다. 

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_architecture_011.png)

- **컨테이너 네트워크 인터페이스 소개** 

컨테이너를 네트워크에 쉽게 연결하기 위해서, 컨테이너 네트워크 인터페이스가 시작됐다. CNI는 쿠버네티스가 어떤 CNI 플러그인이든 설정할수 있게 해준다. 

- 플러그인 목록 
  - Calico
  - Flannel 
  - Romana
  - Weave Net 
  - 그외 기타 

네트워크 플러그인을 설치하는 것은 어렵지 않다. 데몬셋과 다른 자원 리소스를 가지고 있는 YAML을 배포하면 된다.  
이 YAML 파일은 각 플러그인 프로젝트 페이지에서 제공된다. 

## 서비스 구현 방식 

서비스 구현 방식을 이해해보고자 한다. 

### kube-proxy 소개

서비스와 관련된 모든 것은 각 노드에서 동작하는 kube-proxy 프로세스에 의해 처리된다.  
초기에는 kube-proxy가 실제 프록시로서 연결을 기다리다가, 들어온 연결을 위해 해당 파드로 가는 새로운 연결을 생성했을 생성했다. 
나중에는 성능이 더 우수한 iptables 프록시 모드가 이를 대체했다.   

각 서비스가 안정적인 IP 주소와 포트를 얻는다는 것은 알고 있다. 클라이언트는 IP 주소와 포트를 이ㅛㅇㅇ해 서비스에 접속해 사용한다.  
이 IP 주소는 가산이다. 어떠한 네트워크 인터페이스에도 할당되지 않고 패킷이 노드를 떠날 때 네트워크 패킷 안에서 출발지 혹은 도착지 IP 주소로 
표시되지 않는다. 서비스의 주요 핵심 사항은 서비스가 IP와 포트dml 쌍으로 구성된다는 것으로, 서비스 IP 만으로 아무것도 나타내지 않는다. 

### kube-proxy가 iptables를 사용하는 방법 

API 서버에서 서비스를 생성하면, 가상 IP 주소가 바로 할당된다. 곧이어 API 서버는 워커 노드에서 실행 중인 모든 kube-proxy 에이전트에 새로운 서비스가 생성돼됐음을 
통보한다. 각 kube-proxy 는 실행 중인 노드에 해당 서비스 주소로 접근할 수 있도록 만든다. 이것은 서비스의 IP/포트 쌍으로 향하는 패킷을 가로채서, 목적지 주소를 변경해 패킷이 서비스를 지원하는 여러 
파드 중 하나로 리디렉션되로록 하는 몇 개의 iptables 규칙을 설정함으로써 이뤄진다.  

kube-proxy는 API 서버에서 서비스가 변경되는 것을 감지하는 것 외에도, 엔드 포인트 오브젝트가 변경되는 것을 같이 감시한다.   

엔드포인트 오브젝트는 서비스를 지원하는 모든 파드의 IP/포트 쌍을 가지고 있다. 그러므로 kube-proxy 는 모든 엔드 포인트 오브젝트도 감시해야 한다.  
결국 뒷받침 파드가 생성하거나 삭제 될 때, 파드의 레디시느 상태가 바뀔 때 아니면 파드의 레이블이 수정돼 서비스에서 빠지거나 범위를 벗어나느 경우마다  
엔드포인트 오브젝트가 변경된다. 

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_architecture_012.png)

처음 패킷의 목적지는 서비스의 IP와 포트로 지정된다. 패킷이 네트워크로 전송되기 전에 노드 A의 커널이 노드에 설정된 iptables 규칙에 따라 먼저 처리한다.  
커널은 패킷이 iptables 규칙 중에 일치하는게 있는지 검사한다. 그 규칙 중 하나에서 패킷 중 목적지 IP가 172.30.0.1이고 목적지 포트가 80인 포트가 있다면,   
임의로 선택된 파드의 IP와 포트로 교체돼야 한다고 알려준다.  

예제의 패킷은 해당 규칙과 일치하기 때문에, 패킷의 목적지 IP/포트가 변경돼야 한다. 이번 예제에서는 파드 B2가 무작위로 선택됐기 때문에, 패킷의 목적지 IP가  
10.1.2.1 로 포트는 8080으로 변경된다. 여기부터는 클라이언트 파드가 서비스를 통하지 않고 패킷을 파드 B로 직접 보내는 것과 같다. 

## 고가용성 클러스터 진행 

### 애플리케이션 가용성 높이기 

쿠버네티스에서 애플리케이션을 실행할 때, 다양한 컨트롤러는 노드 장애가 발생해도 애플리케이션이 특정 규모로 원활하게 동작할수 있게 해준다.  
애플리케이션의 가용선을 높이려면 이플로이먼트 리소스로 애플리케이션을 실행하고 적절한 수의 레플리카를 설정하기만 하면 되며, 나머지는 쿠버네티스가 처리한다. 

- 가동 중단 시간을 줄이기 위한 다중 인스턴스 실행 

가동 중단 시간을 줄이기 위해서는 애플리케이션을 수평으로 확장할 수 있어야 하지만 애플리케이션이 그런 경우에 속하지 않더라도 레플리카 수가 1로 지정된 디플로이먼트를 사용해야 한다.  
레프리카를 사용할 수 없게 되면, 새 레플리카로 빠르게 교체된다. 관련된 컨트롤러가 노드에 장애가 있음을 인지하고 새 파드 레플리카를 생성한 후에 컨테이너를 시작하는 데 시간이 걸린다.  
그 시간에 짧은 중단 시간이 발생하는 것은 어쩔 수없다. 

- 수평 스케일링이 불가능한 애플리케이션을 위한 리더 선출 메커니즘 사용 

중단 시간이 발생하는 것을 필하려면, 활성 복제본과 함쎄 비활성 본제본을 실행해두고 빠른 임대 혹은 리더 선출 메커니즘을 이용해 단 하나만 활성화 상태로 만들어야 한다.  
리더 선출에 익숙하지 않다면 이는 여러 애플리케이션 인스턴스가 분산 환경에서 실행 중인 경우에는 누가 리더가 될지 합의 하는 방법이다. 예를 들어 리더가 되는 것을 기다리는 형태가 있고,   
모든 인스턴스가 활성화 상태이면서 리더만 쓰기를 할 수 있는 유일한 인스턴스이고 리더가 아닌 인스턴스들은 데이터를 읽을 수만 있는 기능을 제공하는 경우가 있다.  

이렇게 하면 경쟁조건으로 에측할 수 없는 시스템 동작이 발생하더라도, 두 인스턴스가 같은 작업을 하지 않도록 할 수 있다. 

이 메커니즘은 애플리케이션 자체에 포함될 필요는 없다. 모든 리더 선출 작업을 수행하고 활성화 될 때 신호철 메인 컨테이너로 보내는 사이드가 컨테이너를 사용할 수 있다.   
쿠버네티스에서 리더 선출에 관련된 예제를 찾을 수 있다. 

### 쿠버네티스 컨트롤 플레인 구성 요소의 가용성 향상 

쿠버네티스 사용성을 높이기 위해서는 다음 구성 요소의 여러 인스턴스를 여러 마스터 노드에서 실행해야 한다. 

- etcd 
- API 서버 
- 컨트롤러 매니저 
- 스케줄러

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_architecture_013.png)

#### etcd 클러스터 실행 

etcd는 분산 시스템으로 설계됐으므로 그 주요 기능은 여러 etcd 인스턴스를 실행하는 기능이라, 가용선을 높이는 것은 큰 문제가 되지 않는다.  
필요한 수의 머신에서 인스턴스를 실행하고 서로를 인식할 수 있게 하면 된다. 이는 모든 인스턴스의 설정에 다른 인스턴스 목록을 포함시켜 이 작업을 수행할 수 있다. 
예를 들어 인스턴스를 시작할 때 다른 etcd 인스턴스에 접근할 수 있는 IP와 포트를 지정한다.    

etcd는 모든 인스턴스에 걸쳐 데이터를 복제하기 때문에 세 대의 머신으로 구성된 클러스터는 한 노드가 실패하더라도 읽기와 쓰기 작업을 모두 수행할 수 있다.  
노드 한 대 이상으로내 결함성을 높이려면 5대 혹은 7개로 구성된 etcd 노드가 필요하다. 5대 인 경우에는 2대, 7대인 경우 3대까지의 실패를 허용할 수 있다. 
7대 이상의 etcd 인스턴스가 필요한 경우는 거의 없으며, 오히려 성능에 영향을 줄 수 있다. 

#### 여러 API 서버 인스턴스 실행 

API 서버의 가용선을 높이는 일은 훨씬 간단하다. API 서버는 상태를 저장하지 않기 때문에 필요한 만큼 API 서버를 실행할 수 있고 서로 인지할 필요도 없다. 일반적으로 
모든 etcd 인스턴스에 API 서버를 함께 띄운다. 이렇게 하면 모든 API 서버가 로컬에 있는 etcd 인스턴스와만 통신하기 때문에, etcd 인스턴스 앞에 노드 밸런서를 둘 필요가 없다. 
반면 API 서버는 로드 밸런스 앞에 위치하기 때문에 클라이언트는 항상 정상적인 API 서버 인스턴스에만 연결된다. 

#### 컨트롤러와 스케줄러의 고가용성 확보 

여러 복제본을 동신에 실행할 수 있는 API 서버와는 달리, 컨트롤러 매니저나 스케줄러의 여러 인스턴스를 동시에 실행하는 것은 쉬운 일이 아니다.  
컨트롤러와 스케쥴러는 클러스터 상태를 감시하고 상태가 변경되 ㄹ때 반응해야 하는데 이런 구성 요소의 여러 인스턴스가 동시에 실행되 같은 동작을 수행하면 클러스터 상태가 예상보다   
더 많이 변경될 가능성이 있끼 때문이다. 여러 인스턴스가 서로 경쟁해 원하지 않는 결과를 초래할 수 있다.   

이런 이유 때문에 컨트롤러 매니저나 스케쥴러 같은 구성 요소는 여러 인스턴스를 실행하기 보다는 한 번에 하나의 인스턴스만 활성화되게 해야 한다.  
다행이 이는 구성 요소 자체적으로 수행된다. 리더만 실제로 작업을 수행하고 나머지 다른 인스턴스는 대기하면서 현재 리더가 실패할 경우를 기다린다.  
리더가 실패한 경우 나머지 인스턴트 중에서 새로운 리더를 선출하고 기존 작업을 계속 이어서 수행한다. 이 메커니즘은 두 구성 요소가 동시에 같은  
작업을 수행하지 않도록 방지한다. 

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_architecture_014.png)

컨트롤러 매니저와 스케줄러는 API 서버 그리고 etcd와 같이 실행되거나 다른 머신에서 실행할 수 있다. 같이 실행될 경우에는 로컬 API 서버와 직접 통신하고  
다른 머신에서 실행될 때는 로드 밸런서 API 서버와 통신한다. 

#### 컨트롤 플레인 구성 요소에서 사용되는 리더 선출 메커니즘 이해 

리더를 선출하기 위해 서로 직접 대화할 필요가 없다는 것이다. 리더 선출 메커니즘은 API 서버에 오브젝트를 생성하는 것만으로 완전히 동작한다. 
또한 특별한 유형의 리소스를 사용하는 것도 아니다. 여기서는 이를 위해 엔드 포인트 리소스를 사용한다.  
 이를 하기 위해 엔드포인트 오브젝트를 사용하는 데에 특별한 이유는 없다. 단지 동일한 이름으로 된 서비스가 존재하지 않는한 부작용이 없기 때문에   
엔드포인트 오브젝트를 사용한다.  
  
```shell
# 모든 스케줄러 인스턴스는 kube-scheduler 엔드 포인트 리소스를 생성하려고 시도한다. 
$ kubectl get endpoints kube-controller -n kube-system -o yaml 
```

낙관적 동시성은 여러 인스턴스가 자신의 이름을 리소스에 기록하려고 노력하지만 단 하나의 인스턴스만 성공한다는 것을 보장한다. 이름 기록 성공 여부에 따라 
각 인스턴스는 리더인지 아닌지 알 수 있다. 리더가 되면 주기적으로 리소스를 갱신해서, 다른 모든 인스턴스에서 리더가 살아 있음을 알 수 있도록 해야 한다. 
리더에 장애가 있으면 다른 인스턴스는 리소스가 한동안 갱신되지 않는 것을 확인하고, 자신의 이름을 리소스에 기록해 리더가 되려도 시도한다. 


# Section 15 

## 인증이해 

API 서버를 하나 이상의 인증 플러그인으로 구성할 수 있다고 했다. API 서버가 요청을 받으면 인증 플러그인 목록을 거치면서 요청이 전달되고, 
각각의 인증 플러그인이 요청을 검사해서 보낸 사람이 누구인가를 밝혀내려 시도한다. 요청에서 해당 정보를 처음으로 추출해낸 플러그인은 사용자 이름,  
사용자 ID와 클라이언트가 속한 그룹을 API 서버 코어를 반환한다. 

- 클라이언트의 인증서 
- HTTP 헤더로 전달된 인증 토큰 
- 기본 HTTP 인증 
- 기타 

### 사용자 그룹  

인증 플러그인은 인증된 사용자의 사용자 이름과 그룹을 반환한다. 쿠버네티스는 해당 정보를 어디에도 저장하지 않는다. 

#### 사용자 

쿠버네티스는 API 서버에 접속하는 두 종류의 클라이언트를 구분한다. 

- 사용자(실제사람)
  - 사용자는 싱글 사인온과 같은 외부 시스템에 의해 관리된다.
  - 사용자 계정의 경우 별도 나타내는 자원은 없기 때문에, API 서버를 통해 사용자를 생성, 업데이트 또는 삭제할 수 없다는 뜻이다.  
- 파드 
  - 파드는 서비스 어카운트라는 메커니즘을 사용하며, 클러스터에 서비스 어카운트 리소스로 생성되고 저장된다. 

#### 그룹 

휴먼 사용자와 서비스 어카운트는 하나 이상의 그룹에 속할 수 있다. 인증 플러그인은 사용자 이름 및 사용자 ID와 함께 그룹을 반환한다고 말했다. 
그룹은 개별 사용자에게 권한을 부여하지 않고 한 번에 여러 사용자에게 권한을 부여하는 데 사용된다.  

- system:unauthenticated 그룹은 어떤 인증 플러그인 에서도 클라이언트를 인증할 수 없는 요청에 사용된다. 
- system:authenticated 그룹은 성공적으로 인증된 사용자에게 자동으로 할당된다. 
- system:serviceaccounts 그룹은 시스템의 모든 서비스어카운트를 포함된다. 
- system:serviceaccounts:<namespace> 는 특정 네임스페이스의 모든 서비스 어카운트를 포함한다. 

### 서비스 어카운트 

- /var/run/secrets/kubernetes.io/serviceaccount/token  

모든 파드는 파드에서 실행 중인 애플리케이션의 아이덴티티를 나타내는 서비스어카운트와 연계돼 있다.  
이 토큰 파일은 서비스 어카운트의 인증 토큰을 갖고 있다. 애플리케이션이 이 토큰을 사용해 API 서버에 접속하면 인증 플러그인이  
서비스 어카운트를 인증하고 서비스어카운트의 사용자 이름을 API 서버 코어로 전달한다. 서비스어카운트의 사용자 이름은 다음과 같은 형식이다. 

```
system:serviceaccount:<namespace>:<service account name> 
```

API 서버는 설정된 인가 플러그인에 이 사용자 이름을 전달하며, 이 인가 플러그인은 애플리케이션이 수행하려는 작업을 서비스 어카운트에서 수행할 수 있는지를 결정한다. 

#### 서비스어카운트 리소스 

서비스 어카운트는 파드, 시크릿, 컨피그맵 등과 같은 리소스이며 개별 네임스페이스로 범위가 지정된다. 각 네임스페이스마다 default 서비스 어카운트가 자동으로 생성된다.  

```shell
$ kubectl get sa
```

default 서비스 어카운트만 갖고 있다. 필요한 경우 서비스 어카운트를 추가할 수 있다. 각 파드는 딱 하나의 서비스와 연계돼 있지만 
여러 파드가 같은 서비스 어카운트를 사용할 수 있다. 파드는 같은 네임스페이스의 서비스어카운트만 사용할 수 있다. 

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_architecture_015.png)

#### 서비스 어카운트가 인가와 어떻게 밀접하게 연계돼 있는지 이해하기 

파드 매니페스트에 서비스 어카운트의 이름을 지정해 파드에 서비스 어카운트 할당할 수 있다. 명시적으로 할당하지 않으면 파드는 네임스페이스에 있는 default 서비스 어카운트를 사용한다.  
파드에 서로 다른 서비스 어카운트를 할당하면 각 파드가 액세스할 수 있는 리소스를 제어할 수 있다. API 서버가 인증 토큰이 있는 요청을 수신하면, API 서버는 토큰을 사용해  
요청을 보낸 클라이언트를 인증한 다음 관련 서비스 어카운트가 요청된 작업을 수행할 수 있는지 여부를 결정한다. API 서버는 클러스터 관리자가 구성한 시스템 전체의 인가 플러그인에서 정보를 얻는다.   


사용가능한 인가 플러그인 중 하나는 역할 기반 액세스 제어 플러그인이며 추후에 설정한다. 

### 서비스 어카운트 생성  

모든 네임스페이스에는 고유한 default 서비스 어카운트가 포함돼 있지만 필요한 경우 추가로 만들 수 있다. 하지만 왜 모든 파드에 default 서비스 어카운트를 사용하지 않고, 
귀찮게 서비스 어카운트를 생성하는 것일까?  

분명한 이유는 클러스터 보안 때문이다. 클러스터의 메타 데이터를 읽을 필요가 없는 파드는 클러스터에 배포된 리소스를 검색하거나 수정할 수 없는 제한된 계정으로 실행해야 한다.  
리소스의 메타 데이터를 검색해야 하는 파드는 해당 오브젝트의 메타 데이터만 읽을 수 이ㅏㅆ는 서비스 어카운트로 실행해야 하며, 오브젝트를 수정해야 하는 파드는 API 오브젝트를  
수정할 수 있는 고유한 서비스 어카운트로 실행해야 한다. 

```shell
$ kubectl create serviceaccount foo 

$ kubectl describe sa goo 

$ kubectl describe secret foo-token-qzq7j
```

#### 서비스 어카운트의 마운트 가능한 시크릿 이해 

kubectl describe 를 사용해 서비스 어카운트를 검사하면 토큰이 Mountable secrets 목록에 표시된다.  
기본적으로 파느는 원하는 모든 시크릿을 마운트 할 수 있다. 그러나 파드가 서비스어카운트를 설정할 수 있다. 이 기능을 사용하려면 서비스어카운트가 다음 어노테이션을 포함하고 있어야 한다. 


``` 
$ kubernetes.io/enforce-mountable-secrets="true" 
```

서비스 어카운트에 이 어노테이션이 달린 경우 이를 사용하는 모든 파드는 서비스 어카운트의 마운트 가능한 시크릿만 마운트 할 수 있다. 

### 파드에 서비스어카운트 할당 

추가 서비스어카운트를 만든 후에는 이를 파드에 할당해야 한다. 파드 정의의 spec.serviceAccountName 필드에서 서비스어카운트 이름을 설정하면 된다.

#### 사용자 정의 서비스어카운트를 사용하는 파드 생성 

tutum/curl 이미지 기반의 컨테이너와 함께 앰배서더 컨테이너를 실행하는 파드를 배포했다. 이 파드를 이용해 API 서버의 REST 인터페이스를 탐색했다. 
앰배서더 컨테이너는 kubectl proxy 프로세스를 실행했으며, 파드의 서비스 어카운트 토큰을 사용해 API 서버를 인증했다 

```yaml
apiVersion: v1 
kind: Pod 
metadata: 
  name: curl-custom-sa 
spec: 
  serviceAccountName: foo 
  containers: 
    - name: main 
      image: tutum/curl 
      command: ["sleep", "9999999"]
    - name: ambassador 
      image: luksa/kubectl-proxy:1.6.2 
```

```shell 
$ kubectl exec -it curl-custom-sa -c main 
```

#### API 서버와 통신하기 위해 사용자 정의 서비스어카운트 토큰 사용 

이 토큰을 사용해 API 서버와 통신할 수 있는지 살펴보자. 

```shell
$ kubectl exec -it curl-custom-sa -c main curl localhost:8001/api/v1/pods 
```

사용자 정의 서비스 어카운트 파드로 나열할 수 있다는 뜻이다. 이는 클러스터가 RBAC 인가 플러그인을 사용하지 않거나, 제시한 지침에 따라   
모든 서비스 어카운트에 모든 권한을 부여했기 때문일 수 있다. 클러스터가 적절한 인가를 사용하지 않는 경우 default 서비스 어카운트만으로도 모든 작업을 수행할 수 있으므로  
추가적인 서비스 어카운트를 생성하고 사용하는 것은 의미가 없다. 이 경우 서비스 어카운트를 사용하는 유일한 이유는 앞에서 설명한대로 마운트 가능한 시크릿을 적용하거나 서비스어카운트를 
이미지 풀 시크릿을 제공하기 위한 것이다. 

## 역할 기반 액세스 제어로 클러스터 보안 

RBAC 인가 플러그인이 GA로 승격됐으며 이제 많은 클러스터에서 기본적으로 할성화돼 있다. RBAC는 권한이 없는 사용자가 클러스터 상태를 보거나  
수정하지 못하게 한다. 디폴트 서비스어카운트는 추가 권한 부여하지 않는한 클러스터 상태를 볼수 없으며 어떤식으로는 수정할수 없다. 

### RBAC 인가 플러그인 소개  

쿠버네티스 API 서버는 인가 플러그인을 사용해 액션을 요청하는 사용자가 액션을 수행할 수 있는지 점검하도록 설정할 수 있다.  
API 서버가 REST 인터페이스를 제공하므로 사용자는 서버에 HTTP 요청을 보내 액션을 수행한다. 사용자는 요청에 자격증명을 표함시켜  
자신을 인증한다. 

#### 액션 이해하기

REST 클라이언트는 GET, POST, PUT, DELETE 및 기타 유형의 HTTP 요청을 특정 REST 리소스를 나타내는 특정 URL 경로로 ㅂ ㅗ낸다. 
쿠버네티스에서 이러한 리소스에는 파드, 서비스, 시크릿 등이 있다. 

- 파드 가져오기 
- 서비스 생성하기 
- 시크릿 업데이트 
- 기타 등등 

이러한 예의 동사는 클라이언트가 수행한 HTTP 메서드에 매핑된다. 명사는 쿠버네티스 리소스와 정확하게 매핑된다. 
API 서버 내에서 실행되는 RBAC와 같은 인가 플러그인은 클라이언트가 요청한 자원에서 요청한 동사를 수행할 수 있는지를 판별한다.  

--- 

전체 리소스 유형에 보안 권한을 적용하는 것 외에도 RBAC 규칙은 특정 리로스 인스턴스에도 적용할 수 있다.  
그리고 리소스가 아닌 URL 경로에도 권한을 적용할 수 있다는 것을 알게 될텐데, 이는 API 서버가 노출하는 모든 경로가 리소스를 매핑한 것은 아니기 때문이다.  

#### RBAC 플러그인 이해 

이름에서 알 수 있듯이 RBAC 인가 플러그인은 사용자 액션을 수행할 수 있는지 여부를 결정하는 핵심 요소를 사용자 롤을 사용한다. 주체는 하나 이상의 롤과 연계돼 있으며 
각 롤은 특정 리소스에 특정 동사를 수행할 수 있다.   
 사용자에게 여러 롤이 잇는 경우 롤에서 허용하는 모든 작업을 수행할 수 있다. 예를 들어 사용자 롤에 시크릿 정보를 업데이트하는 권한이 없으면 API 서버는 사용자가 
시크릿 정보에 대해 PUT 또는 PATCH 요청을 수행하지 못하게 한다.  
RBAC 플러그인으로 인가를 관리하는 것은 간단하다. 

### RBAC 리소스 소개 

- 롤과 클러스터롤 : 리소스에 수행할 수 있는 동사를 지정한다. 
- 롤바인딩과 클러스터롤 바인딩 : 위의 롤을 특정 사용자, 그룹 또는 서비스 어카운트에 바인딩 한다.

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_architecture_016.png)

롤과 클러스터롤 또는 롤바인딩과 클러스터롤바인딩의 차이점은 롤과 롤바인딩은 네임스페이스가 지정된 리소스이고 클러스터롤과 클러스터롤바인딩은 네임 스페이스를 지정하지 않는  
클러스터 수준의 리소스라는 것이다. 하나의 네임스페이스에 여러 롤 바인딩이 존재할 수 있다. 또한 여러 클러스터롤 바인딩과 클러스터롤을 만들 수 있다.    
롤 바인딩이 네임스페이스가 지정됐음에도 네임스페이스가 지정되지 않는 클러스터롤을 참조할 수 있다는 것이다. 

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_architecture_017.png)

#### 실습을 위한 설정  

RBAC 리소스가 API 서버로 수행할 수 잇는 작업에 어떤 영향을 주는지 살펴보기 전에 클라스터에 RBAC가 활성화돼 있는지 확인해야 한다.  
인가 플러그인으로 병렬로 활성화돼 있을 수 있으며 그중 하나라도 액션을 수행하도록 허용하는 경우 액션이 허용된다. 

```shell
$ kubectl delete clusterrolebinding permissive-binding 
``` 

#### 네임스페이스 생성과 파드 실행 

```shell
$ kubectl create ns foo 

$ kubectl run test --image=luksa/kubectl-proxy -n foo 

$ kubectl create ns bar 

$ kubectl run test --image=luksa/kubectl-proxy -n bar 

# 이제 터미널 두 개를 열고 kubectl exec를 사용해 각 파드 내에서 셸을 실행한다. 
$ kubectl get po -n foo 

$ kubectl exec -it text-123123434-ttq36 -n foo sh 

# 파드에서 서비스 목록 나열하기 
# RBAC가 활성화돼 있는 상태에서 파드가 클러스터 상태 정보를 읽을 수 없다는 것을 확인하기 위해 curl를 사용해 foo 네임스페이스의 서비스를 나열한다. 
$ curl localhost:8001/api/v1/namespaces/foo/services 
```

### 롤과 롤바인딩 사용  

롤 리소스는 이떤 리소스에 어떤 액션을 수행할 수 있는지 정의한다. 다음 예제는 사용자 foo 네임 스페이스에서 서비스를 가져오고 나열 할 수 있는 롤을 정의 한다. 

```yaml
apiVersion: rbac.authorization.k8s.io/v1 
kind: Role 
metadata: 
  namespace: foo 
  name: service-reader 
rules: 
  - apiGroups: [""] 
    verbs: ["get", "list"]
    resources: ["services"]
```

- 롤은 네임스페이스가 지정된다 ( 네임스페이스를 생략하면 현재 네임스페이스가 된다 )
- 서비스는 이름이 없는 core apiGroup의 리소스다. 따라서 "" 이다. 
- 개별 서비스의 이름을 가져오고 모든 항목의 나열이 허용된다. 
- 이 규칙(rule)은 서비스와 관련이 있다. 

이 롤 리소스는 foo 네임 스페이스에 생성된다. 각 리소스 유형이 리소스 매니페스트의 apiVersion 필드에 지정한 API 그룹에 속한다는 것을 배웠다. 
롤 정의에서는 각 규칙내에 나열된 리소스의 apiGroup을 지정해야 한다. 여러 API 그룹에 속한 리소스에 관한 액세스를 허용하는 경우 여러 규칙을 사용한다. 

#### 롤 생성하기 

``` 
$ kubectl create -f service-reader.yaml -n foo 
```

GKE를 사용하는 경우 클러스터 관리자 권한이 없기 때문에 위의 명령이 실패할 수 있다. 

``` 
$ kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=your.email@address.com 
```

YAML 파일을 이용해 service-reader 롤을 생성하는 대신 kubectl create role 명령을 사용해 롤을 생성할 수도 있다. 

``` 
$ kubectl create role service-reader --verb=get --verb=list --resource=services -n bar 
``` 

이 두 롤을 사용해 두 파드 내에서 foo와 bar 네임스페이스의 서비스를 나열할 수 있다. 그러나 두 롤을 만드는 것만으로는 충분하지 않다. 각 롤을 네임스페이스의 서비스 어카운트에 바인딩해야 한다. 

#### 서비스어카운트에 롤을 바인딩하기  

롤은 수행할 수 있는 액션을 정의하지만 누가 수행할 수 있는지는 지정하지 않는다. 그렇게 하려면 주체에 바인딩해야 한다. 여기서 주체란 사용자, 서비스 어카운트 혹은 그룹이 될 수 있다.  
주체에 롤을 바인딩하는 것은 롤 바인딩 리소스를 만들어 주생한다. 롤을 default 서버스 어카운드에 바인딩하기 위해 다음 명령을 실행한다. 

``` 
$ kubectl create rolebinding test --role=service-reader --serviceaccount=foo:default -n foo 
``` 

명령은 그 자체로 설명이 가능해야 한다. foo 네임 스페이스의 default 서비스 어카운트에 service-reader  롤을 바인딩하는 롤 바인딩을 만들고 있다. 

> 서비스 어카운트 대신 사용자에게 롤을 바인딩하려면 --user 인수를 사용해 사용자 이름을 지정해라. 그룹을 바인딩하려면 --group를 사용하라. 

```shell
$ kubectl get rolebinding test -n foo -o yaml 
```

보다시피 롤바인딩은 항상 하나의 롤을 참조하지만 여러 주체에 롤을 바인딩할 수 있다.  
이 롤 바인딩은 네임스페이스 foo의 파드가 실행 중인 서비스어카운트에 바인딩하기 때문에 이제 해당 파드 내에서 서비스를 나열할 수 있다. 

``` 
$ curl localhost:8001/api/v1/namespaces/foo/services
```

#### 롤 바인딩에서 다른 네임스페이스의 서비스 어카운트 포함하기 

``` 
$ kubectl edit rolebiding test -n foo 
```

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_architecture_018.png) 

### 클러스터롤과 클러스터롤바인딩 사용하기 

과 롤바인딩은 네임스페이스가 지정된 리소스로, 하나의 네임스페이스 상에 상주하며 해당 네임스페이스의 리소스에 적용된다는 것을 의미하지만 롤 바인딩은 다른 네임 스페이스의 서비스 어카운트도 참조할 수 있다.    
이러한 네임 스페이스가 지정된 리소스 외에도 클러스터롤와 클러스터롤바인딩이라는 두 개의 클러스터 수준의 RBAC 리소스도 있다. 이것들은 네임스페이스를 지정하지 않는다.   


일반 롤은 롤이 위치하고 있는 동일한 네임스페이스의 리소스에만 액세스할 수 있다. 다른 네임스페이스의 리소스에 누군가가 액세스할 수 있게 하려면 해당 네임스페이스마다 롤과 
롤 바인딩을 만들어야 한다. 이를 모든 네임 스페이스로 확장하려면 각 네임스페이스에서 동일한 롤과 롤바인딩을 생성해야 한다. 네임스페이스를 추가로 만들 때마다 이 두 개의 리소스를 
만들어야 한다는 것도 기억해야 한다. 

#### 클러스터 수준 리소스에 액세스 적용 

``` 
# pv-reader 클러스터롤 생성 
$ kubectl create clusterrole pv-reader --verb=get,list --resource=persitenctvolumes 

# 정의된 clusterrole 확인 
$ kubectl get clusterrole pv-reader -o yaml

# 파드내 쉘의 실행 
$ curl localhost:8001/api/v1/persistentvolumes  
# 퍼시스턴트 볼륨은 네임스페이스에 연관되지 않기 때문에 URL에 네임스페이스가 없다. 

# 롤 바인딩 생성 - 클러스터롤에 대한 권한을 받기 위해서 시도함. 
$ kubectl create rolebinding pv-test --clusterrole=pv-reader --serviceaccount=foo:default -n foo 

# 아래의 코드는 동작하지 않음
$ curl localhost:8001/api/v1/persistentvolumes 

# 실제 명세에서는 정상적으로 표기 된다. 
$ kubectl get rolebindings pv-test -o yaml
# 롤 바인딩을 생성하고 클러스터롤을 참조해서 네임스페이스가 지정된 리소스에 액세스 하게 할 수 있지만 클러스터 수준 리소스에는 동일한 방식을 사용할 수 없다. 
# 클러스터 수준 리소스에 액세스 권한을 부여하려면 항상 클러스터 롤 바인딩을 사용해야 한다. 

# 기존의 RoleBinding 삭제 
$ kubectl delete rolebinding pv-test 

# 클러스터 롤 바인딩 작성 
$ kubectl create clusterrolebinding pv-test --clusterrole=pv-reader --serviceaccount=foo:default 

# 퍼시스턴스 볼룸 나열 가능 여부 확인 
$ curl localhost:8001/api/v1/persistentvolumes 
   
```

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_security_001.png)

> 롤 바인딩은 클러스터롤 바인딩을 참조하더라도 클러스터 수준 리소스에 액세스 권한을 부여할 수 없다.

#### 리소스가 아닌 URL에 액세스 허용하기 

```shell 
$ kubectl get clusterrole system:discovery -o yaml 
```

위의 클러스터롤이 리소스 대신 URL을 참조하는 것을 볼 수 있다. verbs 필드는 이러한 URL에 HTTP 메소드를 정의해서 사용할 수 있도록 한다. 

> 리소스가 아닌 URL의 경우 create나 update대신  post, put과 patch가 사용된다. 동사는 소문자로 지정해야 한다. 

클러스터 수준 리소스와 마찬가지로 리소스가 아닌 URL의 클러스터롤은 클러스터롤 바인딩으로 바인딩해야 한다. 롤바인딩으로 바인딩 하면 아무런 효과가 없다. 

```shell
# 기본 system:discovery 클러스터롤 바인딩 
$ kubectl get clusterrolebinding system:discovery -o yaml 
```

> 그룹은 인증 플러그인의 도메인에 있다. API 서버가 요청을 받으면 인증 플러그인을 호출해 사용자가 속한 그룹 목록을 얻는다. 이 정보는 인가에 사용된다. 

파드 내부에서 /api URL 경로를 액세스 해보고, 로컬 컴퓨터에서 인증 토큰을 지정하지 않고 액세스 해봄으로써 이를 확인할 수 있다. 

```shell
$ curl https://$(minikube ip):8443/api -k 
```

이제 클러스터롤과 클러스터 롤 바인딩을 사용해 클러스터 수준 리소스와 리소스가 아닌 URL에 액세스 권한을 부여했다. 이제 네임스페이스가 
지정된 롤 바인딩과 함께 클러스터롤을 사용해 롤바인딩의 네임스페이스에 있는 네임스페이스가 지정된 리소스에 액세스 권한을 부여하는 방법을 살펴보자 

#### 특정 네임 스페이스의 리소스에 액세스 권한을 부여하기 위해서 클러스터롤 사용하기 

 ```shell 
 $ kubectl get clusterrole view -o yaml 
 ```

이 규칙은 ConfigMaps, Endpoints, PersistentVolumeClaims 등과 같은 리소스를 가져오고 나열하고 볼 수 있게 허용된다.   
이들은 네임스페이스가 지정된 리소스이지만, 독자 여러분은 클러스터롤을 보고 있다.  
클러스터롤은 클러스터롤바인딩과 롤 바인딩 중 어디에 바인드되느냐에 따라 다르다. 클러스터 롤 바인딩과 롤 바인딩 중 어디에 바인드되느냐에 따라 다르다. 
클러스터롤 바인딩을 생성하고 클러스터롤을 참조하면, 바인딩에 나열된 주체는 모든 네임 스페이스에 있는 지정된 리소스를 볼 수 있다.  
반면 롤바인딩을 만들면 바인딩에 나열된 주체가 롤 바인딩의 네임스페이스에 있는 리소스만 볼 수 있다.  
 
```shell 
# 내부 pod 호출
$ curl localhost: 8001/api/vl/pods
 
$ curl localhost: 8001/api/vl/namespaces/foo/pods
 
$ kubectl create clusterrolebinding view-test --clusterrole=view --serviceaccount=foo: default
 
$ curl localhost: 8001/api/vl/namespaces/foo/pods
 
$ curl localhost: 8001/api/vl/namespaces/bar/pods 
 
$ curl localhost: 8001/api/vl/pods
```
 
```shell 
$ kubectl delete clusterrolebinding view-test
 
$ kubectl create rolebinding view-test --clusterrole=view  --serviceaccount=foo:default -n foo

# 내부 pod 호출 
$ curl localhost: 8001/api/vl/namespaces/foo/pods
 
$ curl localhost: 8001/api/vl/namespaces/bar/pods
 
$ curl localhost: 8001/api/vl/pods
```

#### 롤, 클러스터롤, 롤바인딩 과 클러스터롤 바인딩 조합에 관한 요약 
 
#### 디폴트 클러스터롤과 클러스터바인딩의 이해 
 
쿠버네티스는 API 서버가 시작될 때마다 업데이트 되는 클러스터롤과 클러스터롤바인딩의 디폴트 세트를 제공한다.  
이렇게 하면 실수로 삭제하거나 최신 버전의 쿠버네티스가 클러스터 롤과 바인딩을 다르게 설정해 사용하더라도 모든 디폴트 롤과 바인딩을 다시 생성되게 한다.   
 
```shell 
$ kubectl get clusterrolebindings 
 
$ kubectl get custerroles 
```
 
##### view 클러스터롤을 사용해 리소스에 읽기 전용 액세스 허용하기 
 
##### edit 클러스터롤을 사용해 리소스에 변경 허용하기 
 
edit 클러스터롤은 네임스페이스 내의 리소스를 수정할 수 있을 뿐만 아니라 시크릿을 읽고 수정할 수도 있다. 
 
##### admin 클러스터롤을 사용해 네임스페이스에 제어 권한 허용하기 
 
네임스페이스에 있는 리소스에 관한 완전한 제어 권한이 admin 클러스터롤에 부여된다. 
이 클러스터롤을 가진 주체는 리소스 쿼터와 네임스페이스 리소스 자체를 제외한 네임 스페이스 내의 모든 리소스를 일고 수정할 수 있다.  
edit와 admin 클러스터롤 간의 주요 자이점은 네임스페이스에서 롤과 롤바인딩을 보고 수정할 수 있다. 

> 권한 상승을 방지하기 위해 API 서버는 사용자가 해당 롤에 나열된 모든 권한(및 동일한 범위)을 가지고 있는 경우에만롤을 만들고 업데이트할수 있다.

##### cluster-admin 클러스터롤을 사용해 완전한 제어 허용하기 

쿠버네티스 클러스터를 완전히 제어하려면 cluster-admin 클러스터롤 주체에 할당하면 된다. 앞서 본것 처럼 admin 클러스터롤에서는 사용자가 네임스페이스의 리소스 쿼터 개체 또는  
네임스페이스 리소스 자체를 수정할 수 없다. 사용자가 이 작업을 수행하도록 허용하려면 cluster-admin 클러스터롤을 참조하는 롤 바인딩을 생성해야 한다. 이것은 롤 바인딩에 포함된  
사용자에게 롤 바인딩이 생성된 네임 스페이스의 모든 측면을 완전하게 제어할 수 있다.  
 
##### 그 밖의 디폴트 클러스터 이해하기 

디폴트 클러스터 목록에는 접두사 system: 으로 시작하는 클러스터롤이 있다. 이들은 다양한 쿠버네티스의 구성 요소에서 사용된다. 그중에서 스케줄러에서 사용되는  
system:kube-scheduler와 kubelet에서 사용되는 system:node 등이 있다. 컨트롤러 매니저는 하나의 파드로 실행되지만 내부에서 실행되는 각 컨트롤러는 별도의  
클러스터롤과 클러스터롤 바인딩을 사용할 수 있다. 
 
### 인가 권한을 현명하게 부여하기 

기본적으로 네임 스페이스의 디폴트 서비스 어카운트에는 인증되지 않은 사용자의 권한 이외에는 어떤 권한도 없다. 따라서 기본적으로 파드는 클러스터 상태조차 볼수 없다.  
이를 수행할 수 있는 적절한 권한을 부여하는 것은 전적으로 사용자의 몫이다. 당연히 모든 서비스 어카운트에 cluster-admin 클러스터롤을 부여하는 것은 좋지 못한 생각이다. 
모든 사람에게 자신의 일을 하는 데 꼭 필요한 권한만 주고, 한 가지 이상의 권한을 주지 않는 것이 가장 좋다. ( 최소 권한의 원칙 )

#### 각 파드에 특정 서비스 어카운트 생성 

각 파드를 위한 특정 서비스 어카운트를 생성한 다음 롤 바인딩으로 맞춤형 롤과 연겨하는 것이 바람직한 접근 방법이다.   
만약 파드를 읽는 기능과 수정기능이 필요한 경우에는 각각 두개의 서비스 어카운트를 만들고, 각 파드 스펙의 serviceAccountName 속성에 이 서비스 어카운트를 설정해 사용하라. 

#### 애플리케이션이 탈취될 가능성을 염두에 두기 

원치 않는 사람이 결국 서비스 어카운트 인증 토큰을 손에 넣을 수 있다는 가능성을 예상해야 하며, 따라서 실제로 피해를 입지 않도록 항상 서비스 어카운트에 제한을 둬야 한다. 

# Section 16 

## 파드에서 호스트 노드의 네임스페이스 사용 
## 컨테이너의 보안 컨텍스트 구성 
## 파드의 보안 관련 기능 사용 제한 
## 파드 네트워크 격리  

# Section 17 

## 파드 컨테이너의 리소스 요청 
## 컨테이너에 사용 가능한 리소스 제한 
## 파드 QoS 클래스 이해  
## 네임스페이스별 파드에 대한 기본 요청과 제한 설정  
## 네임스페이스의 사용 가능한 총 리소스 제한하기 
## 파드 리소스 사용량 모니터링 

# Section 18

## 수평적 파드 오토스케일링 
## 수직적 파드 오토스케일링 
## 수평적 클러스터 노드 확장 

# Section 19 

## 테인트와 톨러레이션을 사용해 특정 노드에서 파드 실행 제한 
## 노드 어피니티를 사용해 파드를 특정 노드로 유인하기 
## 파드 어피니티와 안티-어피니티를 이용해 함께 배치하기 

# Section 20

## 모든 것을 하나로 모아보기 
## 파드 라이프 사이클 이해 
## 모든 클라이언트 요청의 적절한 처리 보장 
## 쿠버네티스에서 애플리케이션을 쉽게 실행하고 관리할 수 있게 만들기 
## 개발 및 테스트 모범 사례 

# Tips

- [kubernetes cheat sheet](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-strong-getting-started-strong-)

