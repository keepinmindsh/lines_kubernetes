# Section 6 - 서비스에 대하여

## Cheat Sheet 

```shell
kubectl create ingress simple --rule="foo.com/bar=svc1:8080,tls=my-cert"
```

```shell
kubectl create ingress catch-all --class=otheringress --rule="/path=svc:port"
```

```shell
kubectl create ingress annotated --class=default --rule="foo.com/bar=svc:port" \
--annotation ingress.annotation1=foo \
--annotation ingress.annotation2=bla
```

```shell
kubectl create ingress multipath --class=default \
--rule="foo.com/=svc:port" \
--rule="foo.com/admin/=svcadmin:portadmin"
```

```shell
kubectl create ingress ingress1 --class=default \
--rule="foo.com/path*=svc:8080" \
--rule="bar.com/admin*=svc2:http"
```

```shell
kubectl create ingress ingtls --class=default \
--rule="foo.com/=svc:https,tls" \
--rule="foo.com/path/subpath*=othersvc:8080"
```

```shell
kubectl create ingress ingsecret --class=default \
--rule="foo.com/*=svc:8080,tls=secret1"
```

```shell
kubectl create ingress ingdefault --class=default \
--default-backend=defaultsvc:http \
--rule="foo.com/*=svc:8080,tls=secret1"
```

## 서비스

쿠버네티스의 서비스는 동일한 서비스를 제공하는 파드 그룹에 지속적인 단일 접점을 만들려고 할 때 생성하는 리소스다. 각 서비스는 서비스가 존재하는 동안 절대 바뀌지 않는 IP 주소와 포트가 있다.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
``` 

> 서비스는 모든 수신 port를 targetPort에 매핑할 수 있다. 기본적으로 그리고 편의상, targetPort는 port 필드와 같은 값으로 설정된다. 

### Service 구성

서비스는 다수 파드 앞에서 로드 밸런서 역할을 한다. 파드가 하나가 있으면 서비스는 이 파드 하나에 정적주소를 제공한다.
서비스를 지원하는 파드가 하나든지 그룹이든지에 관계없이 해당 파드가 클러스터 내에서 이동하면서 생성되고 삭제되며 IP가
변경되지만, 서비스는 항상 동일한 주소를 가진다.

```shell

$ kubectl apply -f helloworld.service.yaml

# 변경 
$ kubectl expose pod lines-cluster --type=LoadBalancer --name lines-admin-front-http

```

- 서비스 조회하기

```shell
kubectl get services
```

### yaml 디스크립터

```yaml
apiVersion: v1
kind: Service
metadata:
  name: lines-admin-nextjs-service
spec:
  type: LoadBalancer
  selector:
    app: lines-admin-nextjs
  ports:
    - port: 3000
      targetPort: 3000
```

이 서비스는 포트 3000의 연결을 허용하고 각 연결을 app=lines-admin-nextjs 와 일치하는 파드의 포트 8080으로 라우팅한다.

- 서비스 확인하기

```shell
$ k get svc
kubernetes                   ClusterIP      10.124.0.1    <none>         443/TCP          100d
lines-admin-nextjs-service   LoadBalancer   10.124.3.83   34.27.67.137   3000:30765/TCP   60d
```

- 실행 중인 컨테이너에 원격으로 명령어 실행

```shell
$ k exec kubia-7nog1 -- curl -s http://10.111.249.153 
```

> 더블 대시를 사용하는 이유  
> 명령어의 더블 대시(--)는 kubectl 명령줄 옵션의 끝을 의미한다. 더블 대시 뒤의 모든 것은 파드 내에서 실행돼야 하는 명령이다.  
> 명령 줄에 대시로 시작하는 인수가 없는 경우 더블 대시를 사용할 필요가 없다.

#### 서비스에 파드를 설정하는 방법 

파드의 포트 정의에 이름이 있으므로, 서비스의 targetPort 속성에서 이 이름을 참조할 수 있다. 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app.kubernetes.io/name: proxy
spec:
  containers:
  - name: nginx
    image: nginx:stable
    ports:
      - containerPort: 80
        name: http-web-svc

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app.kubernetes.io/name: proxy
  ports:
  - name: name-of-service-port
    protocol: TCP
    port: 80
    targetPort: http-web-svc
```

#### 동일한 서비스에서 여러 개의 포트 노출 ( 멀티 포트 서비스 )

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 9376
    - name: https
      protocol: TCP
      port: 443
      targetPort: 9377
```

> 쿠버네티스의 일반적인 이름과 마찬가지로, 포트 이름은 소문자 영숫자와 - 만 포함해야 한다. 포트 이름은 영숫자로 시작하고 끝나야 한다.
> 예를 들어, 123-abc 와 web 은 유효하지만, 123_abc 와 -web 은 유효하지 않다.

#### 서비스의 세션 어피니티 구성

특정 클라이언트의 모든 요청을 매번 같은 파드로 리디렉션하려면 서비스의 세션 어피니티 속성을 기본값 None 대신 ClientIP로 설정한다.  
서비스 프록시는 동일한 클라이언트 IP의 모든 요청을 동일한 파드로 전달한다.

```yaml
apiVersion: v1 
kind: Service 
spec: 
  sessionAffinity: ClientIP 
  ...
```
#### 엔드포인트 슬라이스 - 셀렉터가 없는 서비스   

서비스는 일반적으로 셀렉터를 이용하여 쿠버네티스 파드에 대한 접근을 추상화하지만, 셀렉터에 대신 매칭되는 엔드포인트슬라이스 오브젝트와 함께 사용되면,  
다른 종류의 백엔드도 추상화할 수 있으며, 여기에는 클러스터 외부에서 실행되는 것도 포함된다. 

- 프로덕션 환경에서는 외부 데이터베이스 클러스터를 사용하려고 하지만, 테스트 환경에서는 자체 데이터 베이스를 사용한다. 
- 한 서비스에서 다른 네임스페이스 또는 다른 클러스터의 서비스를 지정하려고 한다. 
- 워크로드를 쿠버네티스로 마이그레이션하고 있다. 해당 방식을 평가하는 동안, 쿠버네티스에서는 백엔드의 일부만 실행한다.
  
이 서비스에는 셀렉터가 없으므로, 매칭되는 엔드포인트슬라이스 (및 레거시 엔드포인트) 오브젝트가 자동으로 생성되지 않는다.   
엔드포인트슬라이스 오브젝트를 수동으로 추가하여, 서비스를 실행 중인 네트워크 주소 및 포트에 서비스를 수동으로 매핑할 수 있다.  

```yaml
apiVersion: discovery.k8s.io/v1
kind: EndpointSlice
metadata:
  name: my-service-1 # 관행적으로, 서비스의 이름을
                     # 엔드포인트슬라이스 이름의 접두어로 사용한다.
  labels:
    # "kubernetes.io/service-name" 레이블을 설정해야 한다.
    # 이 레이블의 값은 서비스의 이름과 일치하도록 지정한다.
    kubernetes.io/service-name: my-service
addressType: IPv4
ports:
  - name: '' # 9376 포트는 (IANA에 의해) 잘 알려진 포트로 할당되어 있지 않으므로
             # 이 칸은 비워 둔다.
    appProtocol: http
    protocol: TCP
    port: 9376
endpoints:
  - addresses:
      - "10.4.5.6" # 이 목록에 IP 주소를 기재할 때 순서는 상관하지 않는다.
      - "10.1.2.3"
```

서비스를 위한 엔드포인트슬라이스 오브젝트를 생성할 때, 엔드포인트슬라이스 이름으로는 원하는 어떤 이름도 사용할 수 있다. 
네임스페이스 내의 각 엔드포인트슬라이스 이름은 고유해야 한다. 해당 엔드포인트슬라이스에 kubernetes.io/service-name 레이블을 설정하여 
엔드포인트슬라이스를 서비스와 연결할 수 있다.

> 엔드포인트 IP는 루프백(loopback) (IPv4의 경우 127.0.0.0/8, IPv6의 경우 ::1/128), 또는 링크-로컬 (IPv4의 경우 169.254.0.0/16와 224.0.0.0/24, IPv6의 경우 fe80::/64)이 되어서는 안된다.
> 엔드포인트 IP 주소는 다른 쿠버네티스 서비스의 클러스터 IP일 수 없는데, kube-proxy는 가상 IP를 목적지(destination)로 지원하지 않기 때문이다.



#### 환경 변수를 통한 서비스 검색

```yaml
$ k exec lines-admin-nextjs-deployment-864f94fc8d-j4bkj env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=lines-admin-nextjs-deployment-864f94fc8d-j4bkj
NODE_VERSION=19.3.0
YARN_VERSION=1.22.19
NODE_ENV=production
PORT=3000
KUBERNETES_SERVICE_HOST=10.124.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT_443_TCP_PROTO=tcp
LINES_ADMIN_NEXTJS_SERVICE_SERVICE_HOST=10.124.3.83
LINES_ADMIN_NEXTJS_SERVICE_PORT_3000_TCP_PROTO=tcp
KUBERNETES_PORT=tcp://10.124.0.1:443
KUBERNETES_PORT_443_TCP_PORT=443
LINES_ADMIN_NEXTJS_SERVICE_PORT_3000_TCP=tcp://10.124.3.83:3000
LINES_ADMIN_NEXTJS_SERVICE_PORT_3000_TCP_PORT=3000
LINES_ADMIN_NEXTJS_SERVICE_PORT_3000_TCP_ADDR=10.124.3.83
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP=tcp://10.124.0.1:443
KUBERNETES_PORT_443_TCP_ADDR=10.124.0.1
LINES_ADMIN_NEXTJS_SERVICE_SERVICE_PORT=3000
LINES_ADMIN_NEXTJS_SERVICE_PORT=tcp://10.124.3.83:3000
HOME=/home/bloguser
```

#### DNS를 통한 서비스 검색

파드가 내부 DNS 서버에서 DNS 항목을 가져오고 서비스 이름을 알고 있는 클라이언트 파드는 환경 변수 대신 FQDN으로 액세스 할 수 있다.

#### FQDN을 통한 서비스 연결

```
backend-database.default.svc.cluster.local 
```

backend-database는 서비스 이름이고 default는 서비스가 정의된 네임스페이스를 나타내며, svc.cluster.local은 모든 클러스터의 로컬 서비스 이름에 사용되면 클러스터의 도메인 접미사이다.

> 클라이언트는 여전히 서비스의 포트 번호를 알아야 한다. 서비스가 표준 포트( 80, 5432 )를 사용하는 경우 문제가 되지 않는다.  
> 그렇지 않은 경우 클라이언트는 환경 변수에서 포트 번호를 얻을 수 있어야 한다.

서비스에 연결하는 것은 훨씬 간단한다. 프론트엔드 파드가 데이터베이스 파드와 동일한 네임스페이스에 있을 경우 svc.cluster.local 접시마와 네임스페이스는 생략할 수 있다.
따라서 서비스를 단순히 backend-database라고 할수 있다.

#### 파드 컨테이너 내에서 셸 실행

> 이 작업을 수행하려면 컨테이너 이미지에서 셸 바이너리 실행 파일이 사용 가능해야 한다.

```shell
$ k exec -it {pod name} bash 

$ curl http://kubia.default.svc.cluster.local 

$ curl http://kubia.default 

$ curl http://kubia 

# 위처럼 요청 url에서 서비스 이름을 사용해 서비스에 엑세스 할 수 있다. 각 파드 컨테이너 내부의 DNS Resolver가 구성돼 있기 때문에 네임스페이스와 svc.cluster.local 접미사를 생략할 수 있다. 
# 컨테이너에서 /ect/resolv.conf 파일을 보면 이해할 수 있다. 

$ cat /etc/resolv.conf
```

#### 서비스에 핑을 할수 없는 이유

```shell
$ ping kubia 
```

서비스로 curl은 동작하지만 핑은 응답이 없다. 이는 서비스의 클러스터 IP가 가장 IP이므로 서비스 포트와 결합된 경우에만 의미가 있기 때문이다.

### 클러스터 외부에 있는 서비스 연결

쿠버네티스 서비스 기능으로 외부 서비스를 노출하려는 경우가 있을 수 있다. 서비스가 클러스터내에 있는 파드로 연결을 전달하는 게 아니라, 외부 IP와 포트로 연결을 전달하는 것이다.

#### 서비스 엔드포인트

서비스는 파드에 직접 연결되지 않는다. 대신 엔드포인트 리소스가 그 사이에 있다.

```shell
$  k describe svc lines-admin-nextjs-service
Name:                     lines-admin-nextjs-service
Namespace:                default
Labels:                   <none>
Annotations:              cloud.google.com/neg: {"ingress":true}
Selector:                 app=lines-admin-nextjs  # 서비스의 파드 셀렉터는 엔드포인트 목록을 만드는 데 사용한다. 
Type:                     LoadBalancer
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.124.3.83
IPs:                      10.124.3.83
LoadBalancer Ingress:     34.27.67.137
Port:                     <unset>  3000/TCP
TargetPort:               3000/TCP
NodePort:                 <unset>  30765/TCP
Endpoints:                10.120.0.11:3000,10.120.3.17:3000   # 이 서비스의 엔드 포인트를 나타내는 파드 IP와 포트 목록 
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

엔드포인트 리소스는 서비스로 노출되는 파드의 IP 주소와 포트 목록이다. 엔드 포인트 리소스는 다른 쿠버네티스 리소스와 유사하므로 kubectl get을 사용해 기본 정보를 표시할 수 있다.

```shell
$ k get endpoints lines-admin-nextjs-service
NAME                         ENDPOINTS                           AGE
lines-admin-nextjs-service   10.120.0.11:3000,10.120.3.17:3000   60d
```

#### 서비스 엔드 포인트 수동 구성

파드 셀렉터 없이 서비스를 만들변 쿠버네티스는 엔드포인트 리소스를 만들지 못한다. 파드 셀렉터가 없어, 서비스에 포함된 파드가 무엇인지 알 수 없다.

##### 셀렉터 없이 서비스 생성

```yaml
apiVersion: v1 
kind: Service 
metadata: 
  name: external-service # 서비스 이름은 엔드포인트 오브젝트 이름과 일치해야 한다.  
spec:   # 이 서비스에는 셀렉터가 정의되어 있지 않다. 
  ports:
    - port: 80 
```

##### 셀렉터가 없는 서비스에 관한 엔드포인트 리소스 생성

엔드 포인트는 별도의 리소스 이며, 서비스 속성은 아니다. 셀렉터가 없는 서비스를 생성했기 때문에 엔드포인트 리소스가 자동으로 생성되지 않는다.

```yaml
apiVersion: v1 
kind: Endpoints 
metadata: 
  name: external-service # 엔드포인트 오브젝트의 이름은 서비스 이름과 일치해야 한다. 
subsets: 
  - addresses: 
    - ip: 11.11.11.11 # 서비스가 연결을 전달할 엔드 포인트의 IP 
    - ip: 22.22.22.22
    ports:
    - port: 80 # 엔드 포인트의 대상 포트  
      
```

엔드 포인트 오브젝트는 서비스와 이름이 같아야 하고 서비스를 제공하는 대상 IP 주소와 포트 목록을 가져야 한다. 서비스와 엔드 포인트 리소스가 모두 서버에 게시되면 파드 셀렉터가 있는
일반 서비스 처럼 서비스를 사용할 수 있다. 서비스가 만들어진 후 만들어진 컨테이너에는 서비스의 환경 변수가 포함되며 IP:포트 쌍에 대한 모든 연결은 서비스 엔드 포인트 간에 로드 밸런싱 한다.

> 엔드포인트 대신 엔드포인트슬라이스 API를 사용하는 것을 권장한다.

#### 외부 서비스를 위한 별칭 생성

##### External 별칭 생성

서비스를 사용하는 파드에서 실제 서비스 이름과 위치가 숨겨져 나중에 externalName 속성을 변경하거나 유형을 다시 ClusterIP로 변경하고 서비스 스펙을 만들어
서비스 스펙을 수정하면 나중에 다른 서비스를 가리킬 수 있다.

ExternalName 서비스는 DNS 레벨에서만 구현된다. 서비스에 관한 간단한 CNAME DNS 레코드가 생성된다.

```yaml
apiVersion: v1 
kind: Service 
metadata: 
  name: external-service
spec:
  type: ExternalName 
  externalName: someapi.somecompany.com
  ports:  
  - port: 80
```

### 외부 클라이언트에 서비스 노출

- 노드 포트로 서비스 유형 설정
- 서비스 유형을 노드 포트로 유형의 확장인 로드밸런서로 설정
- 단일 IP 주소로 여러 서비스를 노출하는 인그레스 리소스 만들기

#### 노드 포트 서비스

```yaml
apiVersion: v1 
kind: Service 
metadata:
  name: kubia-nodeport 
spec: 
  type: NodePort 
  ports:
    - port: 80 
      targetPort: 8080 
      nodePort: 30123 
  selector:
    app: kubi 
```

```shell
$ k get svc kubia-nodeport
```

##### 노드포트 서비스에 액세스 할 수 있도록 방화벽 규칙 변경

```shell
gcloud copute firewall-rules create kubia-svc --allow=tcp:30123 
```

#### 로드밸런서 서비스 생성

```yaml
apiVersion: v1 
kind: Service 
metadata: 
  name: kubia-loadbalancer 
spec:
  type: LoadBalancer 
  ports:
  - port: 80
    targetPort: 8080 
  selector:
    app: kubia
```

```shell
$ k get svc kubia-loadbalancer 
```

#### 외부 연결 특성의 이해

```yaml
sepc:
  externalTrafficPolicy: Local
```

### 자신의 IP 주소 선택 

서비스 생성시 고유한 클러스터 IP 주소를 지정할 수 있다. 이를 위해, .spec.clusterIP 필드를 설정한다.  
예를 들어, 재사용하려면 기존 DNS 항목이 있거나, 특정 IP 주소로 구성되어 재구성이 어려운 레거시 시스템인 경우이다. 

선택한 IP 주소는 API 서버에 대해 구성된 service-cluster-ip-range CIDR 범위 내에 유효한 IPv4 또는 IPv6 주소여야 한다.  
유효하지 않은 clusterIP 주소 값으로 서비스를 생성하려고 하면, API 서버는 422 HTTP 상태 코드를 리턴하여 문제점이 있음을 알린다. 

### 인그레스 리소스로 서비스 외부 노출

인그레스 : 들어가거나 들어가는 행위, 들어갈 권리, 틀어갈 수단이나 장소, 진입로

#### 인그레스가 필요한 이유

로드밸런서 서비스는 자신의 공용 IP 주소를 가진 로드 밸런서가 필요하지만, 인그레스는 한 IP 주소로 수십 개의 서비스에 접근이 가능하도록 지원해준다.
클라이언트가 HTTP 요청을 인그레스에 보낼 때, 요청한 호스트와 경로에 따라 요청을 전달할 서비스가 결정된다.

인그레스는 네트워크 스택의 어플리케이션 계층에서 작동하며 서비스가 할 수 없는 쿠키 기반 세션 어피니티 등과 같은 기능을 제공할 수 있다.

#### 인그레스 컨트롤러가 필요한 경우

인그레스 오브젝트가 제공하는 기능을 살펴보기 전에 인그레스 리소스를 작동시키려면 클러스터에 인그레스 컨트롤러를 실행해야 한다. 쿠버네티스 환경마다 다른 컨트롤러 구현을
사용할 수 있지만 일부는 기본 컨트롤러를 전혀 제공하지 않는다.

##### Minikube에서 인그레스 애드온 활성화

```shell
$ minikube addons list 

$ minikube addons enable ingress 

$ k get po --all-namespaces 
```

#### 인그레스 리소스 생성

```yaml
apiVersion: extensions/v1beta1
kind: Ingress 
metadata: 
  name: kubia 
spec: 
  rules: 
  - host: kubia.example.com 
    http: 
    paths:
      - path: / 
        backend: 
          serviceName: kubia-nodeport 
          servicePort: 80
``` 

> 클라우드 공급자의 인그레스 컨트롤러는 인그레스가 노드포트 서비스를 가리킬 것을 요구한다. 하지만 그것이 쿠버네티스 자체의 요구사항은 아니다

##### 인그레스로 서비스 액세스

```shell
$ k get ingresses
```

##### 인그레스 컨트롤러가 구성된 호스트의 IP를 인그레스 엔드포인트로 지정

IP를 알고나면 kubia.example.com을 해당 IP로 확인하도록 DNS 서버를 구성하거나, 다음 줄을 /etc/hosts에 추가할 수 있다.

##### 인그레스로 파드 액세스

```shell
$ curl http://kubia.example.com 
You've hit kubia-ke823
```

##### 인그레스 동작 방식

- 클라인트가 kubia.example.com 을 찾는다.
- 클라이언트는 헤더 Host:kubia.example.com 과 함께 HTTP GET 요청을 보낸다.
- 컨트롤러가 파드에 요청을 보낸다.

> 인그레스 컨트롤러는 요청을 서비스로 전달하지 않는다. 파드를 선택하는 데만 사용한다. 모두는 아니지만 대부분의 컨트롤러는 이와 같이 동작한다.

#### 하나의 인그레스로 여러 서비스 노출

인그레스 스펙을 보면 규칙과 경로가 모두 배열이므로 여러 항목을 가질 수 있다.

##### 동일한 호스트의 다른 경로로 여러 서비스 매핑

- kubia.example.com/kubia 으로의 요청은 kubia 서비스로 라우팅된다.

```yaml
... 
  - host: kubia.example.com 
    http: 
      paths: 
      - path: /kubia 
        backend: 
          serviceName: kubia 
          servicePort: 80 
      - path: /bar 
        backend: 
          serviceName: bar 
          servicePort: 80 
```

위의 경우 요청은 URL의 경로에 따라 두 개의 다른 서비스로 전송된다. 따라서 클라이언트는 단일 IP 로 두 개의 서비스에 도달할 수 있다.

##### 서로 다른 호스트로 서로 다른 서비스에 매핑하기

```yaml
spec:
  rules:
  - host: foo.example.com 
    http: 
      paths:
      - path: /
        backend:
          serviceName: foo 
          servicePort: 80 
  - host: bar.example.com 
    http: 
      paths:
      - path: / 
        backend: 
          serviceName: bar 
          servicePort: 80
```

#### TLS 트래핑을 처리하도록 인그레스 구성

##### 인그레스를 위한 TLS 인증서 생성

클라이언트가 인그레스 컨트롤러에 대한 TLS 연결을 하면 컨트롤러는 TLS 연결을 종료한다. 클라이언트와 컨트롤러 간의 통신은 암호화 되지만 컨트롤러와 벡엔드 파드 간의
통신은 암호화 되지 않는다. 파드에서 실행 중인 애플리케이션은 TLS를 지원할 필요가 없다. 예를 들어 파드가 웹 서버를 실행할 경우 HTTP 트래픽만 허용하고 인그레스 컨트롤러가 TLS와
관련된 모든 것을 처리하도록 할 수 있다. 컨트롤러가 그렇게 하려면 인증서와 개인 키를 인그레스에 첨부해야 한다.

```shell
$ openssl genrsa -out tls.key 2048 
$ openssl req -new -x509 -key tls.key -out tls.cert -day 360 -subj 

# 만들어진 파일로 시크릿을 만든다. 
$ kubectl create secret tls tls-secret --cert=tls.cert --key=tls.key 
```

> CertificateSigningRequest 리소스로 인증서 서명
> 인증서를 직업 서명하는 대신 CSR 리소스를 만들어 인증서에 서명할 수 있다. 사용자 또는 해당 애플리케이션이 인반 인증서 요청을 생성할 수 있고 CSR에 넣으면
> 그 다음 운영자나 자동화된 프로세스가 다음과 같이 요청을 승인할 수 있다.
> kubectl certificate approve <name of the CSR>

```yaml
  apiVersion: extension/v1beta1
kind: Ingress

metadata:
  name: kubia
spec:
  tls:
  - hosts: 
  - kubia.example.com 
  secretName: tls-secret 
  rules:
  - host: kubia.exampele.com 
  http:
  paths:
  - path: / 
  backend:
  serviceName: kubia-nodeport 
  servicePort: 80  
```

- 전체 TLS 구성이 이 속성 아래에 있다.
- kubia.example.com 호스트 이름의 TLS 연결이 허용된다.
- 개인키와 인증서는 이전에 작성한 tls-secret을 참조한다.

```shell
$ curl -k -v https://kubia.example.com/kubia 
# 명령어의 출력에는 애플리케이션의 응답과 인그레스에 구성한 서버 인증서가 표시된다. 
```

### 파드가 연결을 수락할 준비가 됐을 때 신호 보내기

#### Readiness Prove

레디니스 프로브는 주기적으로 호출되며 특정 파드가 클라이언트 요청을 수신할 수 있는지 결정한다. 컨테이너의 레디니스 프로브가 성공을 반환하면 컨테이너가 요청을 수락할
준비가 됐다는 신호다. 준비가 됐다라는 표시는 분명히 각 컨테이너 마다 다르다. 쿠버네티스는 컨테이너에서 실행되는 애플리케이션이 간단한 GET / 요청에 응답하는지 또는
특정 URL 경로를 호출 할 수 있는지 확인하거나 필요에 따라 애플리케이션이 준비됐는지 확인하기 위해 전체적인 항목을 검사하기도 한다.

##### Types

- 프로세스를 실행하는 Exec 프로브는 컨테이너의 상태를 프로세스의 종료 상태 코드로 결정한다.
- HTTP GET 프로브는 HTTP GET 요청을 컨테이너로 보내고 응답의 HTTP 상태 코드를 보고 컨테이너가 준비됐는지 여부를 결정한다.
- TCP 소켓 프로브는 컨테이너의 지정된 포트로 TCP 연결을 연다. 소켓이 연결되면 컨테이너가 준비된 것으로 간주한다.

라이브니스 프로브와 달리 컨테이너가 준비 상태 점검에 실패하더라도 컨테이너가 종료되거나 다시 시작되지 않는다. 이는 라이브니스 프로브와 레디니스 프로브 사이의 중요한 차이다.
라이브 프로브는 상태가 좋지 않은 컨테이너를 제거하고 새롭고 건강한 컨테이너로 교체해 파드의 상태를 정상으로 유지하는 반면, 레디니스 프로브는 요청을 처리할 준비가 된 파드의 컨테이너만 요청을 수진하도록 한다.
이건은 컨테이너를 시작할 때 주로 필요하지만 컨테이너가 작동한 후에도 유용하다.

###### 레디니스 프로브가 중요한 이유

파드 그룹이 다른 파드에서 제공하는 서비스에 의존한다고 가정해보자. 프론트엔드 파드 중 하나에 연결 문제가 발생해 더 이상 데이터베이스에 연결할 수 없는 경우,
해당 시점에 파드가 해당 요청을 처리할 준비가 되지 않았다는 신호를 레디니스 프로브가 쿠버네티스에게 알리는 것이 현명할 수 있다.  
다른 파드 인스턴스에 동일한 유형의 연결 문제가 발생하지 않는다면 정상적으로 요청을 처리할 수 있다. 레디니스를 사용하면 클라이언트가 정상 상태인 파드하고만 통신하고 시스템에 문제가 있다는 것을 절대 알아차리지 못한다.

#### 파드에 레디니스 프로브 추가

##### 파드 템플릿에 레디니스 프로브 추가

```shell
# 기존 레플리케이션 컨트롤러의 파드 템플릿의 프로브를 추가한다. 
$ kubectl edit rc kubia 
```

```yaml
apiVersion: v1 
kind: ReplicationController 
```

##### 실제 샘플

> [Readiness & Liveness Prove](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: goproxy
  labels:
    app: goproxy
spec:
  containers:
    - name: goproxy
      image: registry.k8s.io/goproxy:0.1
      ports:
        - containerPort: 8080
      readinessProbe:
        tcpSocket:
          port: 8080
        initialDelaySeconds: 5
        periodSeconds: 10
      livenessProbe:
        tcpSocket:
          port: 8080
        initialDelaySeconds: 15
        periodSeconds: 20
```

```shell
$ k get po 
# k exec 명령어로 파드의 컨테이너 내에 touch 명령어가 실행되게 되면 해당 시점에 Pod가 준비상태로 표시된다. 
$ k exec kubia-2r1qp -- touch /var/ready 

$ k get po kubia-2r1qb 
```

#### 실제 환경에서 레디니스 프로브가 수행해야 하는 기능

> 서비스에서 파드를 수동으로 추가하거나 제거하려면 파드와 서비스의 레이블 셀렉터에 enabled=true 레이블을 추가한다.
> 서비스에서 파드를 제거하려면 레이블을 제거하라.

##### 레디니스 프로브를 항상 정의 하라

파드에 레디니스 프로브를 추가하지 않으면 파드가 시작하는 즉시 서비스 엔드 포인트가 된다.
애플리케이션 수신 연결을 시작하는데 너무 오래 걸리는 경우 클라이언트의 서비스 요청은 여전히 시작단계로 수신연결은 수락할 준비가 되지 않은 상태에서
파드로 전달된다. 따라서 클라이언트는 Connnection Refuxed 유형의 에러를 보게 된다.

> 기본 URL에 HTTP 요청을 보내더라도 항상 레디니스 프로브를 정의해야 한다.

##### 레디니스 프로브에 파드의 종료 코드를 포함하지 마라.

쿠버네티스는 파드를 삭제하자마자 모든 서비스에서 파드를 제거하기 때문에 파드 종료 코드를 포함해서는 안된다.!

### 헤드리스 서비스로 개별 파드 찾기

#### 헤드리스 서비스 생성

때때로 로드-밸런싱과 단일 서비스 IP는 필요치 않다. 이 경우, "헤드리스" 서비스라는 것을 만들 수 있는데.  
명시적으로 클러스터 IP(.spec.clusterIO)에 "None"을 지정한다. 

서비스 스펙의 ClusterIP 필드를 None으로 설정하면 쿠버네티스는 클라이언트가 서비스의 파드에 연결할 수 있는 클러스터 IP를 할당하지 않기 때문에 서비스가 헤드리스 상태가 된다.   

```yaml
apiVersion: v1 
kind: Service 
metadata: 
  name: kubia-headless 
spec:
  clusterIp: None 
  ports: 
  - port: 80 
    targetPort: 8080 
  selector: 
    app: kubia 
```

- 셀렉터가 있는 경우 

셀렉터를 정의하는 헤드리스 서비스의 경우, 쿠버네티스 컨트롤 플레인은 쿠버네티스 API 내에서 엔드포인트 슬라이스 오브젝트를 생성하고,  
서비스 하위 파드등을 직접 가리키는 A 또는 AAAA 레코드를 반환하도록 DNS 구성을 변경한다.  

- 셀렉터가 없는 경우 

셀렉터를 정의하지 않는 헤드리스 서비스의 경우, 쿠버네티스 컨트롤 플레인은 엔드포인트슬라이스 오브젝트를 생성하지 않는다.  
하지만, DNS 시스템은 다음 중 하나를 탐색한 뒤 구성한다.

**type: externalName**서비스에 대한 DNS CNAME 레코드 
**externalName**이외의 모든 서비스 타입에 대해, 서비스의 활성 엔드 포인트의 모든 IP 주소에 대한 DNS A / AAAA 레코드   

- IPv4 엔드포인트에 대해, DNS 시스템은 A 레코드를 생성한다. 
- IPv6 엔트포인트에 대해, DNS 시스템은 AAAA 레코드를 생성한다. 


#### DNS로 파드 찾기

##### YAML 매니페스트를 쓰지 않고 파드 실행

```shell
$ k run dnsutils --image=tutum/dnsutils --generator=run-pod/v1 --command --sleep infinity
```

##### 헤드리스 서비스를 위해 반한된 DNS A 레코드

```shell
$ k exec dnsutils nslookup kubia-headless 
```

```shell
$ k exec dnsutils nslookup kubia 
```

#### 모든 파드 검색 - 준비 되지 않은 파드도 포함

```yaml
kind: Service 
metadata:
  annotations:
    service.alpha.kubernetes.io/tolerate-unready-endpoints: "true"
```

### 서비스 문제 해결

- 먼저 외부가 아닌 클러스터 내에서 서비스의 클러스터 IP에 연결되는지 확인한다.
- 서비스에 액세스할 수 있는지 확인하려고 서비스 IP로 핑을 할 필요 없다.
- 레디니스 프로브를 정으했다면 정의했다면 성공했으니 확인하라. 그렇지 않으면 파드는 서비스에 포함되지 않는다.
- 파드가 서비스의 일부인지 확인하려면 k get endpoints 를 사용해 해당 엔드 포인트 오브젝트를 확인한다.
- FQDN이나 그 일부  서비스에 액세스 하려고 하는데 동작하지 않는 경우, FQDN 대신 클러스터 IP를 사용해 액세스 할 수 있는지 확인한다.
- 대상 포트가 아닌 서비스로 노출된 포트에 연결하고 있는지 확인한다.
- 파드 IP에 직접 연결해 파드가 올바른 포트에 연결돼 있는지 확인한다.
- 파드 IP로 애플리케이션에 액세스 할 수 없는 경우 애플리케이션이 로컬 호스트에만 바인딩하고 있는지 확인한다.


> [Kubernetes - Service](https://kubernetes.io/ko/docs/concepts/services-networking/service/)