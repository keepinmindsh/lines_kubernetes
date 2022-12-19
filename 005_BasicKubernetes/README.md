# Kubernetes - Hello World 

- [Link 바로가기](https://github.com/keepinmindsh/lines_kubernetes/tree/main/005_BasicKubernetes/001)

# Kubernetes의 이해

- [Link 바로가기](https://github.com/keepinmindsh/lines_kubernetes/tree/main/005_BasicKubernetes/002)

# Kubernetes - Objects 

- [Pod 에 대한 Link 바로가기](https://github.com/keepinmindsh/lines_kubernetes/tree/main/005_BasicKubernetes/003)
- [Service 에 대한 Link 바로가기](https://github.com/keepinmindsh/lines_kubernetes/tree/main/005_BasicKubernetes/004)
- [Volume 에 대한 Link 바로가기](https://github.com/keepinmindsh/lines_kubernetes/tree/main/005_BasicKubernetes/005)
- [ConfigMap/Secret 에 대한 Link 바로가기](https://github.com/keepinmindsh/lines_kubernetes/tree/main/005_BasicKubernetes/006)
- [Dynamic Provisioning 에 대한 Link 바로가기](https://github.com/keepinmindsh/lines_kubernetes/tree/main/005_BasicKubernetes/007)
- [Namespace, ResourceQuota, LimitRange 에 대한 Link 바로가기](https://github.com/keepinmindsh/lines_kubernetes/tree/main/005_BasicKubernetes/008)
- [Controller 에 대한 Link 바로가기](https://github.com/keepinmindsh/lines_kubernetes/tree/main/005_BasicKubernetes/008)


# Access to the Kubernetes API 

![Access Kubernetes API](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/access_kubernetes_api.png)

## Authentication

### X509 Certs, kubectl, ServiceAccount 

![Access Kubernetes API](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/authentication.png)


##### Service Account 

- kubernetes의 service account 생성하기

```shell 

kubectl apply -f - <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: build-robot
EOF

```

- Service Account 명 생성 규칙 

**DNS Subdomain Names**

Most resource types require a name that can be used as a DNS subdomain name as defined in [RFC 1123](https://tools.ietf.org/html/rfc1123). This means the name must:

1. contain no more than 253 characters.     
2. contain only lowercase alphanumeric characters, '-' or '.'     
3. start with an alphanumeric character.    
4. end with an alphanumeric character.  

- 실제 생성된 계정을 확인해보면, 

```shell

kubectl get serviceaccounts/build-robot -o yaml

```

- Token를 생성하려면, ( Kubectl - Dashboard  로그인 시에도 사용 가능 )

```shell 

kubectl create token build-robot

```

- 오랫동안 유지될 수 잇는 API Token을 만들어야 하는 경우 

```shell 

kubectl apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: build-robot-secret
  annotations:
    kubernetes.io/service-account.name: build-robot
type: kubernetes.io/service-account-token
EOF

```

- token 정보를 확인하려면,  

```shell 

kubectl get secret/build-robot-secret -o yaml

```

- secret 정보 설명을 확인하려면, 

```shell 

kubectl describe secrets/build-robot-secret

```

- 실제 정보를 조회한 결과 

```shell 

kubectl describe secrets/default-secret
Name:         default-secret
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: default
              kubernetes.io/service-account.uid: 629a131a-6adc-460a-b61a-9207deead3d2

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1099 bytes
namespace:  7 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IjZBNU5HVnlKX1owaWU4NG9wTktVSldqQ0p5U241a0FocFNUWFV0cE9majgifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRlZmF1bHQtc2VjcmV0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImRlZmF1bHQiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI2MjlhMTMxYS02YWRjLTQ2MGEtYjYxYS05MjA3ZGVlYWQzZDIiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZGVmYXVsdDpkZWZhdWx0In0.YBFz3B4UgCQHJUyKSxDagFjGKLqirBOGjUCRo_7YRznxGP982ldYf6_-UeTkCoDA8sFZ7YiL5GJICOqmb1Hut2Qi6casvURfHs0LmAPVxefF5-WRcOpkiT-KLjuBE7vhBlgU_sXwE4bHIsxeS0afYCydDRHxRQj0QkCEnTK2UEzHP0S3yLzNHGQscYU9CEv5ZzggxSaL7xf9In2RdazezDlhUw2R9PMjXM6pMDyPK6-gLIV-XZSuhnc11CNboYQdeKvzy7sLLddwxvBlhPurqFtwVeEfnSbv2SmuSApneYsdGk6362rcvDbM0pHK4xebMmBpaa487wIU62q6QMlZqQ

```


### RBAC, Role, RoleBinding Detail 

![Access Kubernetes API](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/authrization001.png)

# Kubernetes Dashboard 구성 

- ClusterRole 
  - ClusterRoleBinding 

- kubeconfig 
  - Client crt
  - Client Key 

- kubernetes-metrics-scraper 

# StatefulSet 

![Stateful Set](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/statefulset.png)

## Stateless Application

- 볼륨이 반드시 필요하지 않음. 
- 요청에 대해서 분산이 될 수 있도록 처리 

### ReplicaSet

## Stateful Application

- 각각의 역할에 따라서 Volume 각각 나뉘어지는 경우가 많음
- 각 App의 역할에 맞게 요청이 수신이 들어와야함, Stateless와 분산의 개념이 다름. 

### StatefulSet 

- Ordinal Index 로 순차적으로 Pod가 생성됨. 
  - 중간에 Pod가 삭제가되면 동일한 이름으로 인덱스가 세팅이 됩니다.

# Ingress 

![Stateful Set](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/ingress.png)

![Stateful Set](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/ingress_controller.png)

- 서비스 로드 밸런싱 
  - 각 서비스의 로드 밸런싱을 담당함. 각각의 Pod에 연결된 서비스에 맞게 보내주는 역할을 함. 
- Canary Upgrade 
  - 서비스 배포시 버전1의 앱과 버전2의 앱 사이의 배포시에 트래픽을 잉그레스를 통해서 조정한다. 

## Ingress Controller 

- Nginx, Kong 

# Autoscaler - HPA 

![HPA](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/hpa.png)

- HPA 
  - HPA의 리소스를 체크하고 있다가 컨트롤러의 ReplicaSet를 증가시키는 작업을 할 수 있다. 
    - Scale Out 
    - Scale In 
  - 기동이 빠르게 되는 App 
  - Stateless App  

![HPA Detail](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/hpa_detail.png)

- VPA
  - StateFul App에 대해서 수직적으로 리소스를 증가시키는 작업을 수행함. 
    - Scale Up 
    - Scale Down 
- CA 
  - 동적으로 워커노드를 추가시킬 경우, Node를 추가하여 Pod를 이동하여 동작할 수 있도록 하는 것! 

## HPA 
