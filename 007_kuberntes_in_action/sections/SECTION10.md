# Section 10 - 시크릿에 대하여 

## Secret

시크릿은 키-값 쌍을 가진 맵으로 컨피그맵과 매우 유사하다. 시크릿은 컨피그 맵과 같은 방식으로 사용할 수 있다.
시크릿은 암호, 토큰 또는 키와 같은 소량의 중요한 데이터를 포함하는 오브젝트이다. 이를 사용하지 않으면 중요한 정보가 파드 명세나  
컨테이너 이미지에 포함될 수 있다. 시크릿을 사용한다는 것은 사용자의 기밀 데이터를 애플리케이션 코드에 넣을 필요가 없음을 뜻한다. 

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

### 시크릿 생성하기 

- kubectl 사용하기 
- 환경 설정 파일 사용하기 
- kustomize 도구 사용하기 

### 개별 시크릿 제한 

개별 시크릿의 크기는 1MiB로 제한된다. 이는 API 서버 및 kubelet 메모리를 고갈시킬 수 있는 매우 큰 시크릿의 생성을 방지하기 위함이다.  
그러나, 작은 크기의 시크릿을 많이 만드는 것도 메모리를 고갈 시킬수 있다. 

### 시크릿 편집 

```shell 
kubectl edit secrets <secret-name>
```

```yaml
# 아래 오브젝트를 편집하길 바란다. '#'로 시작하는 줄은 무시될 것이고,
# 빈 파일은 편집을 중단시킬 것이다. 이 파일을 저장하는 동안 오류가 발생한다면
# 이 파일은 관련된 오류와 함께 다시 열린다.
#
apiVersion: v1
data:
  password: UyFCXCpkJHpEc2I9
  username: YWRtaW4=
kind: Secret
metadata:
  creationTimestamp: "2022-06-28T17:44:13Z"
  name: db-user-pass
  namespace: default
  resourceVersion: "12708504"
  uid: 91becd59-78fa-4c85-823f-6d44436242ac
type: Opaque
```

### 시크릿 사용하기 

시크릿은 데이터 볼륨으로 마운트 되거나 파드의 컨테이너에서 사용할 환경변수로 노출될 수 있다. 또한, 시크릿은 파드에 직접 노출되지 않고, 시스테므이 다른 부분에서도 사용할 수 있다.  
예를 들어, 시크릿은 시스템의 다른 부분이 사용자를 대신해서 외부 시스템과 상호 작용하는데 사용해야하는 자격 증명을 보유할 수 있다.  

특정된 오브젝트 참조가 실제로 시크릿 유형의 오브젝트를 가리키는지 확인하기 위해, 시크릿 볼륨 소스의 유효성이 검사된다.   
따라서, 시크릿은 자신에 의존하는 파드보다 먼제 생성되어야 한다.  

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
      optional: false # 기본값임; "mysecret" 은 반드시 존재해야 함
```

- 특정 경로에 대한 시크릿 키 투영 

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
      items:
      - key: username
        path: my-group/my-username
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


```yaml 
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  USER_NAME: YWRtaW4=
  PASSWORD: MWYyZDFlMmU2N2Rm
```

```shell 
kubectl apply -f mysecret.yaml
```

```yaml 
apiVersion: v1
kind: Pod
metadata:
  name: secret-test-pod
spec:
  containers:
    - name: test-container
      image: registry.k8s.io/busybox
      command: [ "/bin/sh", "-c", "env" ]
      envFrom:
      - secretRef:
          name: mysecret
  restartPolicy: Never
```

```shell
kubectl create secret generic ssh-key-secret --from-file=ssh-privatekey=/path/to/.ssh/id_rsa --from-file=ssh-publickey=/path/to/.ssh/id_rsa.pub
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

#### 시크릿 보륨의 도트 파일 

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

#### 서비스 어카운트 토큰 시크릿 

Kubernetes.io/service-account-token 시크릿 타입은 서비스 어카운트를 확인하는 토큰 자격증명을 저장하기 위해서 사용한다.

```yaml 
apiVersion: v1
kind: Secret
metadata:
  name: secret-sa-sample
  annotations:
    kubernetes.io/service-account.name: "sa-name"
type: kubernetes.io/service-account-token
data:
  # 사용자는 불투명 시크릿을 사용하므로 추가적인 키 값 쌍을 포함할 수 있다.
  extra: YmFyCg==
```

#### 도커 컨피그 시크릿 

이미지에 대한 도커 레지스트리 접속 자격 증명을 저장하기 위한 시크릿을 생성하기 위해서 다음의 type 값 중 하나를 사용할 수 있다.

- kubernetes.io/dockercfg
- kubernetes.io/dockerconfigjson

```yaml 
apiVersion: v1
kind: Secret
metadata:
  name: secret-dockercfg
type: kubernetes.io/dockercfg
data:
  .dockercfg: |
    "<base64 encoded ~/.dockercfg file>"  
```

#### 기본 인증(Basic authentication) 시크릿

kubernetes.io/basic-auth 타입은 기본 인증을 위한 자격 증명을 저장하기 위해 제공된다. 이 시크릿 타입을 사용할 때는 시크릿의 data 필드가 다음의 두 키 중 하나를 포함해야 한다.

- username: 인증을 위한 사용자 이름
- password: 인증을 위한 암호나 토큰

위의 두 키에 대한 두 값은 모두 base64로 인코딩된 문자열이다. 물론, 시크릿 생성 시 stringData 를 사용하여 평문 텍스트 콘텐츠(clear text content)를 제공할 수도 있다.
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-basic-auth
type: kubernetes.io/basic-auth
stringData:
  username: admin      # kubernetes.io/basic-auth 에서 요구하는 필드
  password: t0p-Secret # kubernetes.io/basic-auth 에서 요구하는 필드
```

#### SSH 인증 시크릿

이 빌트인 타입 kubernetes.io/ssh-auth 는 SSH 인증에 사용되는 데이터를 저장하기 위해서 제공된다. 이 시크릿 타입을 사용할 때는 ssh-privatekey 키-값 쌍을 사용할 SSH 자격 증명으로 data (또는 stringData) 필드에 명시해야 할 것이다.

```yaml 
apiVersion: v1
kind: Secret
metadata:
  name: secret-ssh-auth
type: kubernetes.io/ssh-auth
data:
  # 본 예시를 위해 축약된 데이터임
  ssh-privatekey: |
     MIIEpQIBAAKCAQEAulqb/Y ...
```

#### TLS 시크릿 

쿠버네티스는 일반적으로 TLS를 위해 사용되는 인증서 및 관련된 키를 저장하기 위한 빌트인 시크릿 타입 kubernetes.io/tls 를 제공한다.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-tls
type: kubernetes.io/tls
data:
  # 본 예시를 위해 축약된 데이터임
  tls.crt: |
    MIIC2DCCAcCgAwIBAgIBATANBgkqh ...    
  tls.key: |
    MIIEpgIBAAKCAQEA7yn3bRHQ5FHMQ ... 
```

#### 시크릿을 불변으로 지정하기

```yaml
apiVersion: v1
kind: Secret
metadata:
  ...
data:
  ...
immutable: true
```

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

```shell 
$ kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: mycontainer
    image: redis
    env:
      - name: SECRET_USERNAME
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: username
            optional: false # 기본값과 동일하다
                            # "mysecret"이 존재하고 "username"라는 키를 포함해야 한다
      - name: SECRET_PASSWORD
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: password
            optional: false # 기본값과 동일하다
                            # "mysecret"이 존재하고 "password"라는 키를 포함해야 한다
  restartPolicy: Never
EOF 
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