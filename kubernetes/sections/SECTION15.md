- [Section 15 - 쿠버네티스 인증의 이해](#section-15---쿠버네티스-인증의-이해)
  - [Cheat Sheet](#cheat-sheet)
  - [인증이해](#인증이해)
    - [사용자 그룹](#사용자-그룹)
      - [사용자](#사용자)
      - [그룹](#그룹)
    - [서비스 어카운트](#서비스-어카운트)
      - [서비스어카운트 리소스](#서비스어카운트-리소스)
      - [서비스 어카운트가 인가와 어떻게 밀접하게 연계돼 있는지 이해하기](#서비스-어카운트가-인가와-어떻게-밀접하게-연계돼-있는지-이해하기)
    - [서비스 어카운트 생성](#서비스-어카운트-생성)
      - [서비스 어카운트의 마운트 가능한 시크릿 이해](#서비스-어카운트의-마운트-가능한-시크릿-이해)
    - [파드에 서비스어카운트 할당](#파드에-서비스어카운트-할당)
      - [사용자 정의 서비스어카운트를 사용하는 파드 생성](#사용자-정의-서비스어카운트를-사용하는-파드-생성)
      - [API 서버와 통신하기 위해 사용자 정의 서비스어카운트 토큰 사용](#api-서버와-통신하기-위해-사용자-정의-서비스어카운트-토큰-사용)
  - [역할 기반 액세스 제어로 클러스터 보안](#역할-기반-액세스-제어로-클러스터-보안)
    - [RBAC 인가 플러그인 소개](#rbac-인가-플러그인-소개)
      - [액션 이해하기](#액션-이해하기)
      - [RBAC 플러그인 이해](#rbac-플러그인-이해)
    - [RBAC 리소스 소개](#rbac-리소스-소개)
      - [실습을 위한 설정](#실습을-위한-설정)
      - [네임스페이스 생성과 파드 실행](#네임스페이스-생성과-파드-실행)
    - [롤과 롤바인딩 사용](#롤과-롤바인딩-사용)
      - [롤 생성하기](#롤-생성하기)
      - [서비스어카운트에 롤을 바인딩하기](#서비스어카운트에-롤을-바인딩하기)
      - [롤 바인딩에서 다른 네임스페이스의 서비스 어카운트 포함하기](#롤-바인딩에서-다른-네임스페이스의-서비스-어카운트-포함하기)
    - [클러스터롤과 클러스터롤바인딩 사용하기](#클러스터롤과-클러스터롤바인딩-사용하기)
      - [클러스터 수준 리소스에 액세스 적용](#클러스터-수준-리소스에-액세스-적용)
      - [리소스가 아닌 URL에 액세스 허용하기](#리소스가-아닌-url에-액세스-허용하기)
      - [특정 네임 스페이스의 리소스에 액세스 권한을 부여하기 위해서 클러스터롤 사용하기](#특정-네임-스페이스의-리소스에-액세스-권한을-부여하기-위해서-클러스터롤-사용하기)
      - [롤, 클러스터롤, 롤바인딩 과 클러스터롤 바인딩 조합에 관한 요약](#롤-클러스터롤-롤바인딩-과-클러스터롤-바인딩-조합에-관한-요약)
      - [디폴트 클러스터롤과 클러스터바인딩의 이해](#디폴트-클러스터롤과-클러스터바인딩의-이해)
        - [view 클러스터롤을 사용해 리소스에 읽기 전용 액세스 허용하기](#view-클러스터롤을-사용해-리소스에-읽기-전용-액세스-허용하기)
        - [edit 클러스터롤을 사용해 리소스에 변경 허용하기](#edit-클러스터롤을-사용해-리소스에-변경-허용하기)
        - [admin 클러스터롤을 사용해 네임스페이스에 제어 권한 허용하기](#admin-클러스터롤을-사용해-네임스페이스에-제어-권한-허용하기)
        - [cluster-admin 클러스터롤을 사용해 완전한 제어 허용하기](#cluster-admin-클러스터롤을-사용해-완전한-제어-허용하기)
        - [그 밖의 디폴트 클러스터 이해하기](#그-밖의-디폴트-클러스터-이해하기)
    - [인가 권한을 현명하게 부여하기](#인가-권한을-현명하게-부여하기)
      - [각 파드에 특정 서비스 어카운트 생성](#각-파드에-특정-서비스-어카운트-생성)
      - [애플리케이션이 탈취될 가능성을 염두에 두기](#애플리케이션이-탈취될-가능성을-염두에-두기)


# Section 15 - 쿠버네티스 인증의 이해

## Cheat Sheet 

- Cluster Role

```shell 

# "get", "list", "watch" 에 대한 실행에 대해서 pod-reader 클러터터롤에 권한 허용하기 
$ kubectl create clusterrole pod-reader --verb=get,list,watch --resource=pods

# 지정된 Resource Name을 활용하여 clusterrole 생성하기 
$ kubectl create clusterrole pod-reader --verb=get --resource=pods --resource-name=readablepod --resource-name=anotherpod

# API Group에 대해서 
$ kubectl create clusterrole foo --verb=get,list,watch --resource=rs.extensions

# Sub Resources에 대하여 
$ kubectl create clusterrole foo --verb=get,list,watch --resource=pods,pods/status

# Non Resources URL에 대하여 
$ kubectl create clusterrole "foo" --verb=get --non-resource-url=/logs/*

# AggregationRule에 대하여 
$ kubectl create clusterrole monitoring --aggregation-rule="rbac.example.com/aggregate-to-monitoring=true"

```

- Cluster Role Binding 

```shell
$ kubectl create clusterrolebinding cluster-admin --clusterrole=cluster-admin --user=user1 --user=user2 --group=group1
```

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