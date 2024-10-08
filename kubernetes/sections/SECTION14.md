- [Section 14 - 쿠버네티스 아키첵처의 이해](#section-14---쿠버네티스-아키첵처의-이해)
  - [아키텍처 이해](#아키텍처-이해)
    - [쿠버네티스 구성 요소의 분산 특성](#쿠버네티스-구성-요소의-분산-특성)
    - [구성 요소가 서로 통신하는 방법](#구성-요소가-서로-통신하는-방법)
    - [개별 구성 요소의 여러 인스턴스 실행](#개별-구성-요소의-여러-인스턴스-실행)
    - [구성 요소 실행 방법](#구성-요소-실행-방법)
    - [쿠버네티스가 etcd를 사용하는 방법](#쿠버네티스가-etcd를-사용하는-방법)
      - [Etcd](#etcd)
        - [key-value store](#key-value-store)
        - [Operate Etcd](#operate-etcd)
        - [Etcd Version](#etcd-version)
        - [Etcd Cluster](#etcd-cluster)
        - [Etcd Manual Installing](#etcd-manual-installing)
        - [Kubeadm 을 이용한 etcd-master 접근](#kubeadm-을-이용한-etcd-master-접근)
    - [API 서버의 기능](#api-서버의-기능)
    - [API 서버가 리소스 변경을 클라이언트에 통보하는 방법 이해](#api-서버가-리소스-변경을-클라이언트에-통보하는-방법-이해)
    - [스케쥴러 이해](#스케쥴러-이해)
    - [컨트롤러 매니저에서 실행되는 컨트롤 소개](#컨트롤러-매니저에서-실행되는-컨트롤-소개)
    - [Kubelet이 하는 일](#kubelet이-하는-일)
    - [쿠버네티스 서비스 프록시의 역할](#쿠버네티스-서비스-프록시의-역할)
    - [쿠버네티스 애드온 소개](#쿠버네티스-애드온-소개)
  - [컨트롤러가 협업하는 방법](#컨트롤러가-협업하는-방법)
    - [이벤트 체인](#이벤트-체인)
    - [클러스터 이벤트 관찰](#클러스터-이벤트-관찰)
  - [실행중인 파드에 관한 이해](#실행중인-파드에-관한-이해)
  - [파드간 네트워킹](#파드간-네트워킹)
    - [네트워킹 동작 방식 자세히 알아보기](#네트워킹-동작-방식-자세히-알아보기)
  - [서비스 구현 방식](#서비스-구현-방식)
    - [kube-proxy 소개](#kube-proxy-소개)
    - [kube-proxy가 iptables를 사용하는 방법](#kube-proxy가-iptables를-사용하는-방법)
  - [고가용성 클러스터 진행](#고가용성-클러스터-진행)
    - [애플리케이션 가용성 높이기](#애플리케이션-가용성-높이기)
    - [쿠버네티스 컨트롤 플레인 구성 요소의 가용성 향상](#쿠버네티스-컨트롤-플레인-구성-요소의-가용성-향상)
      - [etcd 클러스터 실행](#etcd-클러스터-실행)
      - [여러 API 서버 인스턴스 실행](#여러-api-서버-인스턴스-실행)
      - [컨트롤러와 스케줄러의 고가용성 확보](#컨트롤러와-스케줄러의-고가용성-확보)
      - [컨트롤 플레인 구성 요소에서 사용되는 리더 선출 메커니즘 이해](#컨트롤-플레인-구성-요소에서-사용되는-리더-선출-메커니즘-이해)


# Section 14 - 쿠버네티스 아키첵처의 이해 

## 아키텍처 이해

- 쿠버네티스 컨트롤 플레인 ( Master Node )
    - Kube-apiserver 
      - 전체 서비스와 연결되어 외부와 통신이 가능하게 한다. 
    - etcd 분산 저장 스토리지 
      - Etcd Cluster ( Key - Value Model )
    - 스케쥴러 ( Kube-Scheduler )
    - 컨트롤러 매니저 
      - Controller Manager 
      - Node Controller 
      - Replication Controller 
- 워커노드 ( Worker Nodes )
    - Kubelet
      - Master Node 와 상호작용한다. 
    - 쿠버네티스 서비스 프록시 ( kube-proxy )
      - 서로 다른 노드 간에 접근하기 위해서 사용할 수 있다. 
      - Container 간의 통신을 위하여 
    - 컨테이너 런타임 ( Docker, rkt 외 기타 )
- 애드온 구성 요소
    - 쿠버네티스 DNS 서버
    - 대시보드
    - 인그레스 컨트롤러
    - 힙스터
    - 컨테이너 네트워크 인터페이스 플러그인
- Container Runtime 

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

#### Etcd 

모든 오브젝트는 API 서버가 다시 시작되거나 실패하더라도 유지하기 위해서 메니페스트가 영구적으로 저장될 필요가 있음.
이를 위해서 쿠버네티스는 빠르고, 분산해서 저장되며, 일관된 키-값 저장소를 제공하는 etcd를 사용한다.  
쿠버네티스 API 서버 만이 etcd와 직접적으로 통신하는 유일한 구성요소다. 다른 구성 요소는 API로 간접적으로 데이터를 읽거나 쓸수 있다.  

> [https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)   
> [etcd의 사이즈 제한 및 관리에 대한 부분](https://cloud.google.com/kubernetes-engine/docs/concepts/planning-large-clusters#why_plan_for_large_clusters)

##### key-value store 

> 낙관적 동시성 제어에 관하여 검토 해볼 것!

- 리소스를 etcd에 저장하는 방법

##### Operate Etcd

```shell
./etcdctl set key1 value1 

./etcdctl get key1
```

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

##### Etcd Version 

```shell
./etcdctrl --version
```

```shell
./etcdctrl 
```
##### Etcd Cluster 

- Nodes
- Pods 
- Configs 
- Secrets
- Accounts 
- Roles
- Bindings 
- Others 

##### Etcd Manual Installing 

> [https://github.com/etcd-io/etcd](https://github.com/etcd-io/etcd)

실제 binary 파일을 설치해보기 

> [https://github.com/etcd-io/etcd/releases](https://github.com/etcd-io/etcd/releases)

```shell
rm -rf /tmp/etcd-data.tmp && mkdir -p /tmp/etcd-data.tmp && \
  docker rmi gcr.io/etcd-development/etcd:v3.4.27 || true && \
  docker run \
  -p 2379:2379 \
  -p 2380:2380 \
  --mount type=bind,source=/tmp/etcd-data.tmp,destination=/etcd-data \
  --name etcd-gcr-v3.4.27 \
  gcr.io/etcd-development/etcd:v3.4.27 \
  /usr/local/bin/etcd \
  --name s1 \
  --data-dir /etcd-data \
  --listen-client-urls http://0.0.0.0:2379 \
  --advertise-client-urls http://0.0.0.0:2379 \
  --listen-peer-urls http://0.0.0.0:2380 \
  --initial-advertise-peer-urls http://0.0.0.0:2380 \
  --initial-cluster s1=http://0.0.0.0:2380 \
  --initial-cluster-token tkn \
  --initial-cluster-state new \
  --log-level info \
  --logger zap \
  --log-outputs stderr
```


```shell
docker exec etcd-gcr-v3.4.27  /usr/local/bin/etcd --version
docker exec etcd-gcr-v3.4.27  /usr/local/bin/etcdctl version
docker exec etcd-gcr-v3.4.27  /usr/local/bin/etcdctl endpoint health
docker exec etcd-gcr-v3.4.27  /usr/local/bin/etcdctl put foo bar
docker exec etcd-gcr-v3.4.27  /usr/local/bin/etcdctl get foo
```

##### Kubeadm 을 이용한 etcd-master 접근

```shell
kubectl get pods -n kube-system 
```

```shell
kubectl exec etcd-master -n kube-system etcdctl get / --prefix -keys-only
```

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
    - 수용 가능한 노드의 우선 순위를 정하고 점수가 높은 노드를 선택한다. 
    - 만약 여러 노드가 같은 최상위 점수를 가지고 있다면, 파드가 모든 노드에 고르게 배포되도록 라운드-로빈을 사용한다.

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

> [Kubeadm 설치하기](https://kubernetes.io/ko/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

kubeadm의 경우 Kubelet을 자동으로 배포하기 않기 때문에 수동으로 설치해줘야함. 

> [Kubelet, Kubeadm, Kubectl 설치하기](https://velog.io/@rapidrabbit76/kubelet-kubeadm-kubectl-%EC%84%A4%EC%B9%98)

- kubelet 설치 여부 확인하기 

```shell
ps -aux | grep kubelet 
```

### 쿠버네티스 서비스 프록시의 역할

>  [쿠버네티스에서 프락시](https://kubernetes.io/ko/docs/concepts/cluster-administration/proxies/)

Kubelet 외에도, 모든 워커 노드는 클라이언트가 쿠버네티스 API로 정의한 서빗에 연결할 수 있도록 해주는 kube-proxy도 같이 실행한다.
kube-proxy는 서비스의 IP와 포트로 들어온 접속을 서비스를 지원하는 파드 중 하나와 연결시켜준다.  
서비스가 둘 이상의 파드에서 지원되는 경우 프록시 파드 간에 로드 밸런싱을 수행한다.

- 프록시라고 부르는 이유

kube-proxy의 초기 구현은 사용자 공간에서 동작하는 프록시였다. 실제 서버 프로세스가 연결을 수락하고 이를 파드로 전달했다.  
서비스 IP로 향하는 연결을 가로채기 위해 프록시는 iptables 규칙을 설정해 이를 프록시 서버로 전송했다.

kube-proxy는 실제 프록시이기 때문에 그 이름을 얻었지만, 현재는 훨씬 성능이 우수한 구현체에서 iptables 규칙만 사용해 프록시 서버를
거치지 않고 패킷을 무작위로 선택한 백엔드 파드로 선택한다.

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_architecture_005.png)

> [Kube Proxy 설치 방법 포함 - Tistory](https://dodo-devops.tistory.com/46)

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
- Kubelet은 파드의 컨테이너를 실행

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
표시되지 않는다. 서비스의 주요 핵심 사항은 서비스가 IP와 포트의 쌍으로 구성된다는 것으로, 서비스 IP 만으로 아무것도 나타내지 않는다.

### kube-proxy가 iptables를 사용하는 방법

API 서버에서 서비스를 생성하면, 가상 IP 주소가 바로 할당된다. 곧이어 API 서버는 워커 노드에서 실행 중인 모든 kube-proxy 에이전트에 새로운 서비스가 생성돼됐음을
통보한다. 각 kube-proxy 는 실행 중인 노드에 해당 서비스 주소로 접근할 수 있도록 만든다. 이것은 서비스의 IP/포트 쌍으로 향하는 패킷을 가로채서, 목적지 주소를 변경해 패킷이 서비스를 지원하는 여러
파드 중 하나로 리디렉션되로록 하는 몇 개의 iptables 규칙을 설정함으로써 이뤄진다.

kube-proxy는 API 서버에서 서비스가 변경되는 것을 감지하는 것 외에도, 엔드 포인트 오브젝트가 변경되는 것을 같이 감시한다.

엔드포인트 오브젝트는 서비스를 지원하는 모든 파드의 IP/포트 쌍을 가지고 있다. 그러므로 kube-proxy 는 모든 엔드 포인트 오브젝트도 감시해야 한다.  
결국 뒷받침 파드가 생성하거나 삭제 될 때, 파드의 레디니스 상태가 바뀔 때 아니면 파드의 레이블이 수정돼 서비스에서 빠지거나 범위를 벗어나느 경우마다  
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
