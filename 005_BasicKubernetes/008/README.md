# Object - Namespace, ResourceQuota, LimitRange

![Namespace, Resource Quota, Limit Range](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/kubenetes_image01.png)

## Namespace

--- 

- Pod 명이 중복될 수 없음. Namespace 각각에서는 동일한 파드명을 사용할 수 있다.
- 각각의 서로 다른 Namespace 간에서 Pod 는 서로 네트워크 연결이 가능하지만 이 제어는 Namespace Policy 를 이용할 수 있다.

---

Service Accounts 는 쿠버네티스 API 에 의해서 관리되는 사용들이다. 각 계정은 특정한 namespaces 에 연결된다. Service Account 는 Credentials 의 묶음으로 Secrets 로서 저장된다.
그리고 클러스터 내에서 쿠버네티스 API 와 통신/대화하는 것을 허용하는 각각의 Pod 들로 마운트된다.

---

> [Name Spaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)  
> [Name Spaces by resources quota](https://kubernetes.io/docs/concepts/policy/resource-quotas/)  
> [Share a Cluster with Namespaces](https://kubernetes.io/docs/tasks/administer-cluster/namespaces/)  
> [Namespace와 ServiceAccount의 상관관계 설명](https://kubernetes.io/docs/reference/access-authn-authz/authentication/)


### Namespace 생성

현재의 Namespace에서 특정 서비스 어카운트로 생성되는데 아래의 코드와 같이 동작한다.

- Service Account 생성 예제

```shell 

kubectl create serviceaccount jenkins

```

- Token 생성 예제

```shell 

kubectl create token jenkins

```


### Namespace 1

```yaml
apiVersion: v1 
kind: Namespace 
metadata: 
  name: nm-1 
```

```yaml
apiVersion: v1 
kind: Pod 
metadata: 
  name: pod-1 
  namespace: nm-1 
  labels: 
    nm : pod1 
spec: 
  containers:
  ... 
```

### Namespace 2

```yaml
apiVersion: v1 
kind: Namespace 
metadata: 
  name: nm-2 
```

```yaml
apiVersion: v1 
kind: Pod 
metadata: 
  name: pod-1 
  namespace: nm-2 
  labels: 
    nm : pod1 
spec: 
  containers:
  ... 
```

## Resource Quota

```yaml
apiVersion: v1 
kind: ResourceQuota 
metadata:
  name: rq-1 
  namespace: nm-1 
spec: 
  hard: 
    requests.memory: 3Gi 
    limits.memory: 6Gi
```

```yaml
apiVersion: v1 
kind: Pod 
metadata:
  name: pod-2 
spec:
  containers: 
  - name: container 
    image: tmkube/app
  resources:
    requests:
      memory: 2Gi 
    limits:
      memory: 2Gi 
```

## Limit Range

```yaml
apiVersion: v1 
kind: LimitRange 
metadata: 
  name: lr-1
  namespace: nm-1 
spec:
  limits:
    - type: Container 
      min: 
        memory: 1Gi 
      max: 
        memory: 4Gi 
      defaultRequest: 
        memory: 1Gi 
      default: 
        memory: 2Gi 
      maxLimitRequestRatio: 
        memory: 3
```