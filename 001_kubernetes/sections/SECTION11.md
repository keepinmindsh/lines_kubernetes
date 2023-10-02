# Section 11 - 파드 메타데이터와 리소스 액세스 하기

## 파드 메타데이터와 그외의 리소스에 액세스 하기

### Downward API 로 메타데이터 전달

사용자가 데이터를 직접 설정하거나 파드가 노드에 스케줄링돼 실행되기 이전에 이미 알고 있는 데이터에는 적합하다. 그러나 파드의 IP, 호스트 노드 이름 또는 파드 자체의
이름과 같이 실행 시점까지 알려지지 않은 데이터의 경우는 어떨까? 파드의 레이블이나 어노테이션과 같이 어딘가에 이미 설정된 데이터라면 어떨까?  
아마도 동일한 정보를 여러 곳에 반복해서 설정하고 싶지 않을 것이다.

- fieldRef 활용

```yaml 
apiVersion: v1
kind: Pod
metadata:
  name: kubernetes-downwardapi-volume-example
  labels:
    zone: us-est-coast
    cluster: test-cluster1
    rack: rack-22
  annotations:
    build: two
    builder: john-doe
spec:
  containers:
    - name: client-container
      image: registry.k8s.io/busybox
      command: ["sh", "-c"]
      args:
      - while true; do
          if [[ -e /etc/podinfo/labels ]]; then
            echo -en '\n\n'; cat /etc/podinfo/labels; fi;
          if [[ -e /etc/podinfo/annotations ]]; then
            echo -en '\n\n'; cat /etc/podinfo/annotations; fi;
          sleep 5;
        done;
      volumeMounts:
        - name: podinfo
          mountPath: /etc/podinfo
  volumes:
    - name: podinfo
      downwardAPI:
        items:
          - path: "labels"
            fieldRef:
              fieldPath: metadata.labels
          - path: "annotations"
            fieldRef:
              fieldPath: metadata.annotations
```

- resourceFieldRef 

```yaml 
apiVersion: v1
kind: Pod
metadata:
  name: kubernetes-downwardapi-volume-example-2
spec:
  containers:
    - name: client-container
      image: k8s.gcr.io/busybox:1.24
      command: ["sh", "-c"]
      args:
      - while true; do
          echo -en '\n';
          if [[ -e /etc/podinfo/cpu_limit ]]; then
            echo -en '\n'; cat /etc/podinfo/cpu_limit; fi;
          if [[ -e /etc/podinfo/cpu_request ]]; then
            echo -en '\n'; cat /etc/podinfo/cpu_request; fi;
          if [[ -e /etc/podinfo/mem_limit ]]; then
            echo -en '\n'; cat /etc/podinfo/mem_limit; fi;
          if [[ -e /etc/podinfo/mem_request ]]; then
            echo -en '\n'; cat /etc/podinfo/mem_request; fi;
          sleep 5;
        done;
      resources:
        requests:
          memory: "32Mi"
          cpu: "125m"
        limits:
          memory: "64Mi"
          cpu: "250m"
      volumeMounts:
        - name: podinfo
          mountPath: /etc/podinfo
  volumes:
    - name: podinfo
      downwardAPI:
        items:
          - path: "cpu_limit"
            resourceFieldRef:
              containerName: client-container
              resource: limits.cpu
              divisor: 1m
          - path: "cpu_request"
            resourceFieldRef:
              containerName: client-container
              resource: requests.cpu
              divisor: 1m
          - path: "mem_limit"
            resourceFieldRef:
              containerName: client-container
              resource: limits.memory
              divisor: 1Mi
          - path: "mem_request"
            resourceFieldRef:
              containerName: client-container
              resource: requests.memory
              divisor: 1Mi
```

```shell 
# 실제 downward api에 의한 데이터 확인 용도 파드 배포 
kubectl apply -f https://k8s.io/examples/pods/inject/dapi-volume.yaml

# 파드 조회 
kubectl get pods

# 발생 로그 확인 
kubectl logs kubernetes-downwardapi-volume-example

# 실제 파일 생성 여부 확인을 위한 Pod Shell 접속
kubectl exec -it kubernetes-downwardapi-volume-example -- sh

# Label 조회 정보 확인 
/# cat /etc/podinfo/labels
```

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

> [Downard API - out of date](https://kubernetes.io/ko/docs/concepts/workloads/pods/downward-api/)

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

> [컨테이너를 위한 커맨드와 인자 정의하기](https://kubernetes.io/ko/docs/tasks/inject-data-application/define-command-argument-container/)     
> [종속 환경 변수 정의하기](https://kubernetes.io/ko/docs/tasks/inject-data-application/define-interdependent-environment-variables/)   
> [컨테이너를 위한 환경 변수 정의하기](https://kubernetes.io/ko/docs/tasks/inject-data-application/define-environment-variable-container/)      
> [환경 변수로 컨테이너에 파드 정보 노출하기](https://kubernetes.io/ko/docs/tasks/inject-data-application/environment-variable-expose-pod-information/)   
> [파일로 컨테이너에 파드 정보 노출하기](https://kubernetes.io/ko/docs/tasks/inject-data-application/downward-api-volume-expose-pod-information/)   