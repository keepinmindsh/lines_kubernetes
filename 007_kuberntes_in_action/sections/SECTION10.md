# Section 10 - 시크릿에 대하여 

## Secret

시크릿은 키-값 쌍을 가진 맵으로 컨피그맵과 매우 유사하다. 시크릿은 컨피그 맵과 같은 방식으로 사용할 수 있다.

- 환경 변수로 시크릿 항목을 컨테이너에 전달
- 시크릿 항목을 볼륨 파일로 노출

쿠버네티스는 시크릿에 접근해야 하는 파드가 실행되고 있는 노드에만 개별 시크릿을 배포해 시크릿을 안전하게 유지한다.  
또한 노드 자체적으로 시크릿을 항상 메모리에만 저장되게 하고 물리 저장소에 기록되지 않도록 한다. 물리 저장소는 시크릿을
삭제한 후에도 디스크를 완전히 삭제하는 작업이 필요하기 때문이다.

- 민감하지 않고, 일반 설정 데이터는 컨피그맵을 사용한다.
- 본질적으로 민감한 데이터는 시크릿을 사용해 키 아래에 보관하는 것이 필요하다. 만약 설정 파일이 민감한 데이터와 그렇지 않는 데이터를 모두 가지고 있다면 해당 파일을 시크릿 안에 저장해야 한다.

### Secret

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret
      optional: true
```

#### 기본 토큰 시크릿 소개

모든 파드에는 secret 볼륨이 자동으로 연결돼 있다. 이전 kubectl describe 명령어의 출력은 설정되어 있는 시크릿을 참조한다. 시크릿은 리소스이기 때문에 k get secrets 명령어로 목록을 조회하고  
거기서 default-token 시크릿을 찾을 수 있다.

```shell
k get secrets
W0409 17:54:22.142216   13003 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
To learn more, consult https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke
NAME                  TYPE                                  DATA   AGE
default-token-shkhd   kubernetes.io/service-account-token   3      126d
```

위의 명령어는 secrets를 조회하는 명령어다.

이후 secrets 에서 k describe secrets를 통해서 세부 정보를 확인할 수 있다.

```shell
$ k describe secrets default-token-shkhd
W0409 18:01:26.316320   13101 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
To learn more, consult https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke
Name:         default-token-shkhd
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: default
              kubernetes.io/service-account.uid: 61a3fe51-d59f-4017-9c2c-9b38ca7c8678

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1509 bytes
namespace:  7 bytes
token:      ~~~ 
```

시크릿이 갖고 있는 세 가지 항목(ca.crt, namespace, token)은 파드 안에서 쿠버네티스 API 서버와 통신할 때 필요한 모든 것을 나타낸다.  
이상적으로는 애플리케이션이 완전히 쿠버네티스를 인지하지 않도록 하고 싶지만, 쿠버네티스와 직접 대화하는 방법 외에 다른 대안이 없으면 secret 볼륨을 통해  
제공된 파일을 사용한다.

> 기본적으로 default-token 시크릿은 모든 컨테이너에 마운트되지만, 파드 스펙 안에 auto mountService-AccountToken 필드 값을 false 로 지정하거나
> 파드가 사용하는 서비스 어카운트를 false 로 지정해 비활성화 할 수 있다.

#### 시크릿 생성

fortune-serving Nginx 컨테이너가 HTTPS 트래픽을 제공 할 수 있도록 개선해보자. 이를 위해 인증서와 개인 키를 만들어야 한다.
개인 키는 안전하게 유지해야 하므로 개인 키와 인증서를 시크릿에 넣자.

```shell
$ openssl genrsa -out https.key 2048 
$ openssl req -new -x509 -key https.key  -out https.cert -days 3650 -subj /CN=www.kubia-example.com 
$ echo bar > foo 
$ kuectl create secret generic fortune-https --from-file=https.key --from-file-https.cert --from-file=foo 
```

> 여기에서는 일반적인(generic) 시크릿을 작성하지만, kubectl create secret tls 명령을 이용해 tls 시크릿을 생성할 수도 있다.   
> 이렇게 하면 다른 항목 이름으로 시크릿을 생성할 수 있다.

#### 컨피그맵과 시크릿 비교

```shell
$ k get secret fortune-https -o yaml 
```

```shell
$ k get configmap fortune-config -o yaml 
```

> 민감하지 않은 데이터도 시크릿을 사용할 수 있지만, 시크릿의 최대 크기는 1MB로 제한된다.

Base64 인코딩을 사용하는 까닭은 간단하다. 시크릿 항목에 일반 텍스트뿐만 아니라 바이너리 값도 담을수 있기 때문이다.
Base64 인코딩은 바이너리 데이터를 일반 텍스트 형식인 YAML 이나 JSON 안에 넣을 수 있다.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-basic-auth
type: kubernetes.io/basic-auth
stringData:
  username: admin      # required field for kubernetes.io/basic-auth
  password: t0p-Secret # required field for kubernetes.io/basic-auth
```

stringData 필드는 쓰기 전용이다. 값을 설정할 때만 사용 할 수 있다. kubectl get -o yaml 명령으로 시크릿의 YAML 정의를 가져올 때,
stringData 필드는 표시되지 않는다. 대신 stringData 필드로 지정한 모든 항목은 data 항목 아래에 다른 모든 항목처럼 Base64로 인코딩돼  
표시된다.

##### SSH authentication secrets

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-ssh-auth
type: kubernetes.io/ssh-auth
data:
  # the data is abbreviated in this example
  ssh-privatekey: |
          MIIEpQIBAAKCAQEAulqb/Y ...
```

### 파드에서 시크릿 사용

기존의 시크릿을 수정하거나 신규 시크릿을 생성하고 파드에 연결해서 사용할 수 있다.

```shell
$ k edit configmap fortune-config 
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-test-pod
  labels:
    name: secret-test
spec:
  volumes:
  - name: secret-volume
    secret:
      secretName: ssh-key-secret
  containers:
  - name: ssh-test-container
    image: mySshImage
    volumeMounts:
    - name: secret-volume
      readOnly: true
      mountPath: "/etc/secret-volume"
```

```shell
$ k create secret generic test-db-secret --from-literal=username=testuser --from-literal=password=iluvtests
```

아래의 yaml 처럼 Secret을 생성하고 생성된 시크릿을 파드에 마운트 하는 방법에 대해서 가이드 되어 있다.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: dotfile-secret
data:
  .secret-file: dmFsdWUtMg0KDQo=
---
apiVersion: v1
kind: Pod
metadata:
  name: secret-dotfiles-pod
spec:
  volumes:
  - name: secret-volume
    secret:
      secretName: dotfile-secret
  containers:
  - name: dotfile-test-container
    image: registry.k8s.io/busybox
    command:
    - ls
    - "-l"
    - "/etc/secret-volume"
    volumeMounts:
    - name: secret-volume
      readOnly: true
      mountPath: "/etc/secret-volume"
```

또 다른 Pod Yaml 샘플 정의 ( 참고용 )

```yaml
apiVersion: v1 
kind: Pod 
metaData: 
  name: fortune-https
spec: 
  containers: 
  - image: luksa/fortune:env 
    name: html-generator 
    env: 
    - name: INTERVAL 
      valueFrom: 
       configMapKeyRef: 
         name: fortune-config 
         key: sleep-interval 
    volumeMounts: 
    - name: html 
      mountPath: /var/htdocs 
  - image: nginx:alpine 
    name: web-server 
    volumeMounts: 
    - name: html 
      mountPath: /usr/share/nginx/html 
      readOnly: true 
    - name: config 
      mountPath: /etc/nginx/conf.d 
      readOnly: true 
    - name: certs 
      mountPath: /etc/nginx/certs/ 
      readOnly: true 
    ports: 
    - containerPort: 80 
    - containerPort: 443 
  volumes: 
  - name: html 
    emptyDir: {} 
  - name: config 
    configMap: 
      name: fortune-config 
      items: 
      - key: my-nginx-config.conf 
        path: https.conf 
  - name: certs 
    secret: 
      secretName: fortune-https
```

#### Secret 사용 여부 점검

```shell
$ kubectl port-forward fortune-https  8443:443 & 
$ curl http://localhost:8443 -k 
```

#### 시크릿 볼륨을 메모리에 저장하는 이유

secret 볼륨은 시크릿 파일을 저장하는 데 인메모리 파일 시스템을 사용한다. 컨테이너에 마운트된 볼륨을 조회하면 이를 볼 수 있다.

```shell
$ k exec fortune-https -c web-server -- mount | grep certs 
```

tmpfs를 사용하는 이유는 민감한 데이터를 노출시킬 수도 있는 디스크에 저장하지 않기 위해서다.

#### 환경 변수로 시크릿 항목 노출


```yaml
...
 env: 
 - name: FOO_SECRET 
   valueFrom: 
     secretKeyRef: 
       name: fortune-https
       key: foo 
...
```

- 변수는 시크릿 항목에서 설정된다.
- 키를 갖고 있는 시크릿의 이름
- 노출할 시크릿의 키 이름

쿠버네티스에서 시크릿을 환경변수로 노출할 수 있게 해주지만, 이 기능을 사용하는 것이 가장 좋은 방법은 아니다. 애플리케이션은 일반적으로 오류 보고서에 환경변수를  
기록하거나 시작하면서 로그에 환경 변수를 남겨 의도치 않게 시크릿을 노출할 수 있다. 또한 자식 프로세스는 상위 프로세스의 모든 환경변수를 상속받는데, 만약 애플리케이션이
타사 바이너리를 실행할 경우 시크릿 데이터를 어떻게 사용하는지 알 수 있는 방법이 없다.

> 환경변수로 시크릿을 컨테이너에 전달하는 것은 의도치 않게 노출될 수 있기 때문에 심사숙고 해서 사용해야 한다.
> 안전을 위해서는 시크릿을 노출할 때 항상 secret 볼륨을 사용한다.

### 이미지를 가져올 때 사용하는 시크릿 이해

쿠버네티스에서 자격증명을 전달하는 것이 필요할 때가 있다. 이때에도 시크릿을 통해 이뤄진다.

지금까지 사용한 모든 이미지는 공개 이미지 레지스트리에 저장돼 있었기 때문에 이미지를 가져오는데 특별한 자격증명을 필요로 하지 않았다. 하지만
대부분의 조직은 자신들의 이미지를 모든 사람들이 사용하는 것을 원하지는 않기 때문에 프라이빗 이미지 레지스트리를 사용한다.  
파드를 배포할 때 컨테이너 이미지가 프라이빗 레지스트리 안에 있다면, 쿠버네티스는 이미지를 가져오기 위해 필요한 자격증명을 알아야 한다.

#### 도커 허브에서 프라이빗 이미지 사용

- 도커 레지스트리 자격증명을 가진 시크릿 생성
- 파드 매니페스트 안에 imagePullSecrets 필드에 해당 시크릿 참조

```shell
$ k create secret docker-registry mydockerhubsecret \
  --docker-username=myusername --docker-password=mypassword \ 
  --docker-email=my.email@provider.com 
```

generic 시크릿을 생성하는 것과 다르게, docker-registry 형식을 가진 mydockerhub secret 이라는 시크릿을 만든다.   
여기에 사용할 도커 허브 사용자 이름, 패스워드, 이메일을 지정한다. k describe 명령으로 생성한 시크릿을 살펴보면 .dockercfg 항목을   
갖고 있는 것을 볼 수 있다. 이는 홈 디렉터리에 docker login 명령을 실행할 때 생성된 .dockercfg 파일과 동일하다.

#### 파드 정의에서 도커레지스트리 시크릿 사용

```yaml
apiVersion: v1 
kind: Pod 
metadata: 
  name: private-pod 
spec: 
  imagePullSecrets:
  - name: mydockerhubsecret 
  containers: 
  - image: username/private:tag 
    name: main 
```

#### 모든 파드에서 이미지를 가져올 때 사용할 시크릿을 모두 지정할 필요는 없다.

이미지를 가져올 때 사용할 시크릿을 서비스어카운트에 추가해 모든 파드에 자동으로 추가될 수 있게 하는 방법을 배울 것이다. 