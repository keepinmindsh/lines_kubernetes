# Section 9 - 컨피그 맵에 대하여 

## Cheat Sheet 

```shell 

# config map 조회 
$ kubectl get configmaps game-config -o yaml

# bar 라는 폴더를 바탕으로 하는 my-config라는 컨피그 맵을 생성
$ kubectl create configmap my-config --from-file=path/to/bar

# disk 상의 파일명 대신이 특정 키를 사용해서 my-config라는 컨피그 맵을 생성 
$ kubectl create configmap my-config --from-file=key1=/path/to/bar/file1.txt --from-file=key2=/path/to/bar/file2.txt

# key1=config1, key2=config2 를 사용해서 my-config라는 컨피그 맵을 생성 
$ kubectl create configmap my-config --from-literal=key1=config1 --from-literal=key2=config2

# key=value 값 쌍을 가지는 파일로 부터 my-config라는 컨피그 맵을 생성
$ kubectl create configmap my-config --from-file=path/to/bar

# 환경 설정 파일 ( env.file )로 부터 새로운 config map을 생성하는 방법
$ kubectl create configmap my-config --from-env-file=path/to/bar.env
```

## 컨피그맵과 시크릿 : 애플리케이션 설정

컨피그 맵을 사용해 설정 데이터를 저장할지 여부에 관계없이 다음 방법을 통해 애플리케이션을 구성할 수 있다.

- 컨테이너에 명령줄 인수 전달
- 각 컨테이너를 위한 사용자 정의 환경변수 전달
- 특수한 유형의 볼륨을 통해 설정 파일을 컨테이너에 마운트

## 컨테이너에 명령줄 인자 전달

##### ENTRYPOINT 와 CMD 이해

- ENTRYPOINT는 컨테이너가 시작될 때 호출될 명령어를 정의한다.
- CMD는 ENTRYPOINT에 전달하는 인자를 정의한다.

```shell
$ docker run <image>
```

추가 인자를 지정해 Dockerfile 안의 CMD에 정의된 값을 제공한다.

```shell
$ docker run <image> <argument> 
```

##### shell과 exec 형식 간의 차이점

- shell 형식 - 예: ENTRYPOINT node app.js

위와 같이 하면 컨테이너 내부에서 node 프로세스를 직접 실행한다. 컨테이너 내부에서 실행중인 프로세스 목록을 나열해 직접 실행된 것을 볼 수 있다.

- exec 형식 - 예: ENTRYPOINT ["node", "app.js"]

차이점은 내부에서 정의된 명령을 셸로 호출하는지 여부이다.  
shell 프로세스는 불필요하므로 ENTRYPOINT 명령에서 exec 형식을 사용해 실행한다.

```dockerfile
FROM ubuntu:lastest 
RUN apt-get udpate; apt-get -y install fortune 
ADD fortuneloop.sh /bin/fortuneloop.sh 
ENTRYPOINT ["/bin/fortuneloop.sh"]
CMD ["10"]
```

```shell
$ docker build -t docker.io/luksa/fortune:args . 
$ docker push docker.io/fortune:args 

$ docker run -it docker.io/luksa/fortune:args 

$ docker run -it docker.io/luksa/fortune:args 15 
``` 

### 쿠버네티스에서 명령과 인자 재정의

쿠버네티스에서 컨테이너를 정의할 때, ENTRYPOINT와 CMD 둘 다 재정의할 수 있다.

```yaml
kind: Pod 
spec: 
  containers: 
  - image: some/image
    command: ["/bin/command"]
    args: ["arg1","arg2","arg3"]
```

- command : [ENTRYPOINT] 컨테이너 안에서 실행되는 실행 파일
- args : [CMD] 실행 파일에 전달되는 인자

사용자 정의 주기에 따른 fortune 파드 실행

```yaml
apiVersion: v1 
kind: Pod 
metadata: 
  name: fortune2s 
spec: 
  containers: 
  - images: luksa/fortune:args 
    args: ["2"]
    name: html-generator 
    volumeMounts: 
    - name: html 
      mountPath: /var/htdocs 
```

- 스크립트가 2초마다 새로운 fortune 메세지를 생성하도록 인자 지정


여러 인자를 가질 경우 아래와 같은 배열 표기법이 가능하다.

```yaml
args: 
  - foo
  - bar 
  - "15" 
```

> command 와 args 필드는 파드 생성 이후 업데이트 할 수 없다.

### 컨테이너의 환경 변수 설정

> 컨테이너 명령이나 인자와 마차가지로 환경변수 목록도 파드 생성 후에는 업데이트 할 수 없다.

애플리케이션 구성의 요점은 환겨엥 따라 다르거나 자주 변경되는 설정 옵션을 애플리케이션 소스 코드와 별도로 유지하는 것이다.
만약에 파드 정의를 애플리케이션의 소스 코드로 생각한다면, 설정을 파드 정의에서 밖으로 이동시켜야 한다는 것은 명확하다.

##  컨피그맵

쿠버네티스에서는 설정 옵션을 컨피그 맵이라 부르는 별도 오브젝트로 분리할 수 있다. 컨피그맵은 짧은 문자열에서 전체 설정 파일에 이르는 값을 가지는 키/값    
쌍으로 구성된 맵이다. 애플리케이션은 컨피그맵을 직접 읽거나 심지어 존재하는 것도 몰라도 된다. 대신 맵의 내용은 컨테이너의 환경변수 또는 볼륨 파일로 전달된다.

애플리케이션은 컨피그맵을 직접 읽거나 심지어 존재하는 것은 몰라도 된다. 대신 맵의 내용은 컨테이너의 환경변수 또는 볼륨 파일로 전달 된다. 또한 환경 변수는 $(ENV_VAR)  
구문을 사용해 명령줄 인수에서 참조할 수 있기 때문에, 컨피그맵 항목을 프로세스의 명령줄 인자로 전달할 수도 있다.

![](https://keepinmindsh.github.io/lines/assets/img/configmap001.png)

- 서로 다른 환경에서 사용하는 동일한 이름을 가진 두 개의 컨피그맵

> 컨피그맵은 보안 또는 암호화를 제공하지 않는다. 저장하려는 데이터가 기밀인 경우, 컨피그맵 대신 시크릿 또는 추가(써드파티) 도구를 사용하여 데이터를 비공개로 유지하자.

### 컨피그맵과 파드 

컨피그맵을 참조하는 파드 spec 을 작성하고 컨피그 맵의 데이터를 기반으로 해당 파드의 컨테이너를 구성할 수 있다.  
파드와 컨피그 맵은 동일한 네임스페이스에 있어야 한다.  

> [ConfigMap Data of Kubernetes](https://pwittrock.github.io/docs/tasks/configure-pod-container/configure-pod-configmap/)

- kubectl create configmap 명령 사용

```shell
$ k create configmap fortune-config --from-literal=sleep-interval=25 
``` 

> 컨피그맵 키는 유효한 DNS 서브 도메인이어야 한다(영숫자, 대시, 밑줄, 점만 포함 가능). 필요한 경우 점이 먼저 나올 수 있다.

```shell
$ k create configmap myconfigmap --from-literal=foo=bar --from-literal=bar=baz --from-literal=one=two 
```

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-demo
data:
  # property-like keys; each key maps to a simple value
  player_initial_lives: "3"
  ui_properties_file_name: "user-interface.properties"
  # file-like keys
  game.properties: |
    enemy.types=aliens,monsters
    player.maximum-lives=5    
  user-interface.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true  
```

#### 컨피그 맵을 사용하여 파드 내부에 컨테이너를 구성할 수 잇는 네가지 방법

- 컨테이너 커맨드와 인수 내에서 
- 컨테이너에 대한 환경 변수 
- 애플리케이션이 읽을 수 있도록 읽기 전용 볼륨에 파일 추가 
- 쿠버네티스 API를 사용하여 컨피그맵을 읽는 파드내에서 실행할 코드 작성 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-demo-pod
spec:
  containers:
    - name: demo
      image: alpine
      command: ["sleep", "3600"]
      env:
        # 환경 변수 정의
        - name: PLAYER_INITIAL_LIVES # 참고로 여기서는 컨피그맵의 키 이름과
          # 대소문자가 다르다.
          valueFrom:
            configMapKeyRef:
              name: game-demo           # 이 값의 컨피그맵.
              key: player_initial_lives # 가져올 키.
        - name: UI_PROPERTIES_FILE_NAME
          valueFrom:
            configMapKeyRef:
              name: game-demo
              key: ui_properties_file_name
      volumeMounts:
        - name: config
          mountPath: "/config"
          readOnly: true
  volumes:
    # 파드 레벨에서 볼륨을 설정한 다음, 해당 파드 내의 컨테이너에 마운트한다.
    - name: config
      configMap:
        # 마운트하려는 컨피그맵의 이름을 제공한다.
        name: game-demo
        # 컨피그맵에서 파일로 생성할 키 배열
        items:
          - key: "game.properties"
            path: "game.properties"
          - key: "user-interface.properties"
            path: "user-interface.properties"
```

#### configmap를 별도로 쿠버네티스 오브젝트로 정의하고 Mapping 정의하는 방식

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
    configMap:
      name: game-demo
```

#### configmap 을 직접 선언하는 방식

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-demo-pod
spec:
  containers:
    - name: demo
      image: alpine
      command: ["sleep", "3600"]
      env:
        # Define the environment variable
        - name: PLAYER_INITIAL_LIVES # Notice that the case is different here
                                     # from the key name in the ConfigMap.
          valueFrom:
            configMapKeyRef:
              name: game-demo           # The ConfigMap this value comes from.
              key: player_initial_lives # The key to fetch.
        - name: UI_PROPERTIES_FILE_NAME
          valueFrom:
            configMapKeyRef:
              name: game-demo
              key: ui_properties_file_name
      volumeMounts:
      - name: config
        mountPath: "/config"
        readOnly: true
  volumes:
  # You set volumes at the Pod level, then mount them into containers inside that Pod
  - name: config
    configMap:
      # Provide the name of the ConfigMap you want to mount.
      name: game-demo
      # An array of keys from the ConfigMap to create as files
      items:
      - key: "game.properties"
        path: "game.properties"
      - key: "user-interface.properties"
        path: "user-interface.properties"
```

### 파일로 컨피그맵 생성

컨피그 맵에서는 전체 설정 파일 같은 데이터를 통째로 저장하는 것도 가능하다.

```shell
$ kubectl create configmap my-config --from-file=config-file.conf
```

위의 명령을 실행하면, kubectl을 실행한 디렉터리에서 config-file.conf 파일을 찾는다. 그리고
파일 내용을 컨피그 맵의 config-file.conf 키 값으로 저장한다. 물론 키 이름을 직접 지정할 수도 있다.

```shell
$ kubectl create configmap my-config --from-file=customkey=config-file.conf 
```

이 명령은 파일 내용을 customkey라는 키 값으로 저장한다.   

- 컨피그맵을 생성하거나 기존 컨피그맵을 사용한다. 여러 파드가 동일한 컨피그맵을 참조할 수 있다. 
- 파드 정의를 수정해서 .spec.volumes[] 아래에 볼륨을 추가한다. 볼륨 이름은 원하는 대로 정하고, 컨피그맵 오브젝트를 참조하도록 .spec.volumes[].configMap.name 필드를 설정한다. 
- 컨피그 맵이 필요한 각 컨테이너에 .spec.containers[].volumeMounts[]를 추가한다. .spec.containers[].volumeMounts[].readOnly = true 를 설정하고 컨피그 맵이 연결되기를 원하는 곳에 사용하지 않는 디렉터리 이름으로 .spec.containers[].volumeMounts[].mountPath 를 지정한다. 
- 프로그램이 해당 디텔러티에서 파일을 찾도록 이미지 또는 커맨드 라인을 수정한다. 컨피그맵의 data 맵 각 키는 mountPath 아래의 파일 이름이 된다. 

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
    configmap:
      name: myconfigmap
```

### 디렉터리에 있는 파일로 컨피그 맵 생성

각 파일을 개별적으로 추가하는 대신, 디렉터리 안에 있는 모든 파일을 가져올 수도 있다.

```shell
$ kubectl create configmap my-config --from-file=/path/to/dir 
```

이 명령에서 kubectl은 지정한 디렉터리 안에 있는 각 파일을 개별 항목으로 작성한다.
이때 파일 이름이 컨피그맵 키로 사용하기 유효한 파일만 추가한다.

### 다양한 옵션 결합

아래와 같은 다양한 옵션으로 config map 을 생성할 수 있다.

```shell
$ kubectl create configmap my-configmap 
> --from-file=foo.json 
> --from-file=bar=foobar.conf 
> --from-file=config-opts/ 
> --from-literal=some=thing 
```

실제 환경변수를 컨피그맵에서 가져오는 파드 선언

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: registry.k8s.io/busybox
      command: [ "/bin/sh", "-c", "env" ]
      envFrom:
      - configMapRef:
          name: special-config
  restartPolicy: Never
```

### 파드에 존재하지 않는 컨피그맵 참조

파드를 생성할 때 존재하지 않는 컨피그 맵을 지정하면 어떻게 되는지 궁금할 것이다. 쿠버네티스는 파드를
스케줄링하고 그 안에 있는 컨테이너를 실행하려고 시도한다. 컨테이너가 존재하지 않는 컨피그맵을 참조하려고
하면 컨테이너는 시작하는 데 실패 한다. 하지만 참조하지 않는 다른 컨테이너는 정상적으로 시작된다. 그런 다음
컨피그맵을 생성하면 실패했던 컨테이너는 파드를 만들지 않아도 시작된다.

### 컨피그맵의 모든 항목을 한 번에 환경변수로 전달

컨피그맵에 여러 항목이 포함돼 있을 때 각 항목을 일일이 환경변수로 생성하는 일은 지루하고 오류가 발생하기 쉽다.  
다행시 쿠버네티스 버전 1.6 부터는 컨피그 매의 모든 항목을 환경변수로 노출하는 방법을 제공한다.

```yaml
spec: 
  containers: 
  - image: some-image 
  envFrom: 
  - prefix: CONFIG_ 
  configMapRef: 
    name: my-config-map 
```

####  컨피그맵 항목을 명령줄 인자로 전달

이제 컨피그맵 값을 컨테이너 안에서 실행되는 프로세스의 인자로 전달하는 방법을 살펴보자.  
pod.spec.containers.args 필드에서 직접 컨피그맵 항목을 참조할 수는 없지만 컨피그맵 항목을 환경변수로 먼저 초기화 하고
이 변수를 인자로 참조하도록 지정할 수 있다.

```yaml
apiVersion: v1
kind: Pod 
metadata: 
  name: fortune-args-from-configmap 
spec: 
  containers:
  - image: luksa/fortune:args 
    env: 
    - name: INTERVAL
      valueFrom: 
        configMapKeyRef: 
          name: fortune-config 
          key: sleep-interval 
    args: ["$(INTERVAL)"]
```

#### 컨피그 볼륨을 사용해 컨피그 맵 항목을 파일로 노출


```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config
  namespace: default
data:
  SPECIAL_LEVEL: very
  SPECIAL_TYPE: charm
```


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: registry.k8s.io/busybox
      command: [ "/bin/sh", "-c", "ls /etc/config/" ]
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        # Provide the name of the ConfigMap containing the files you want
        # to add to the container
        name: special-config
  restartPolicy: Never
```

#### 볼륨 안에 있는 컨피그 맵 항목 사용

mountPath : 컨피그 맵으로 정의한 볼륨을 특정 Mount Path 로 지정하는 방법이다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: registry.k8s.io/busybox
      command: [ "/bin/sh","-c","cat /etc/config/keys" ]
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: special-config
        items:
        - key: SPECIAL_LEVEL
          path: keys
  restartPolicy: Never
```

일반적으로 리눅스에서 파일시스템을 비어 있지 않는 디렉터리에 마운트할 때 발생한다. 해당 디렉터리는 마운트한 파일 시스템에 있는 파일만 포함하고,
원래 있던 파일은 해당 파일 시스템이 마운트 돼 있는 동안 접근할 수 없게 된다. 일반적으로 중요한 파일을 포함하는 /etc 디렉터리에 볼륨을 마운트한다고 상상해보자.
/etc 디렉터리에 있어야 하는 모든 원본 파일이 더이상 존재하지 않기 때문에 전체 컨테이너가 손상될 수 있다. 만약 /etc 디렉터리와 같은 곳에
파일을 추가하는 것이 필요하다면, 이 방법을 사용할 수 없다.

##### 특정파일이 마운트된 컨피그맵 항목을 가진 파드

[SubPath의 활용](https://kubernetes.io/docs/concepts/storage/volumes/#using-subpath-expanded-environment)

subPath의 속성은 모든 종류의 볼륨을 마운트할 때 사용할 수 있다. 전체 볼륨을 마운트 하는 대신에 일부만을 마운트 할 수 있다.
하지만 개별 파일을 마운트 하는 이 방법은 파일 업데이트와 관련해서 상대적으로 큰 결함을 가지고 있다.

```yaml
spec: 
  containers: 
  - image: some/image 
    volumeMounts: 
    - name: myvolume 
      mountPath: /etc/someconfig.conf 
      subPath: myconfig.conf 
```

#### 컨피그맵 볼륨 안에 있는 파일 권한 설정

기본적으로 컨피그맵 볼륨의 모든 파일 권한은 644로 설정된다. 파일 권한 설정을 변경하고자 한다면,

```yaml
volumes: 
- name: config 
  configMap: 
    name: fortune-config 
    defaultMode: "6600"
```

### 애플리케이션을 재시작하지 않고 애플리케이션 설정 업데이트

> 컨피그 맵을 업데이트 한 후에 파일이 업데이트 되기 까지 놀라울 정도로 오랜 시간이 걸릴 수 있음을 다시 한 번 강조하고 싶다.

#### 컨피그맵 편집

컨피그 맵을 변경하고 파드 안에서 실행 중인 프로세스가 컨피그맵 볼륨에 노출된 파일을 다시 로드하는 방법을 살펴보자. 이전 Nginx 설정 파일을 편집해
파드 재시작 없이 Nginx가 새설정을 사용하도록 만들자. kubectl edit 명령으로 fortune-config 컨피그맵을 편집해 gzip 압축을 해제하자.

```shell
$ kubectl edit configmap fortune-config 
```

위의 방법으로 파일이 업데이트 되려먼 시간이 걸린다. 결국에는 변경된 설정 파일을 볼 수 있지만, Nginx에는 아무런 영향이 없는 것을 알게 된다.

설정을 다시 로그하기 위해서 Nginx에 신호 전달

```shell
$ kubectl exec fortune-configmap-volume -c web-server -- nginx -s reload 
```

#### 파일이 한꺼번에 업데이트 되는 방법 이해하기

모든 파일이 한 번에 동시에 업데이트 된다. 쿠버네티스는 심볼릭 링크를 사용해 이를 수행한다. 만약 마운트된 컨피그맵 볼륨의 모든 파일을
조회하면 아래와 같은 내용을 확인할 수 있다. 아래의 shell 내용을 보면 마운트된 컨피그 맵 볼륨 안의 파일은 ..data 디렉터리의 파일을 가리키는
심볼릭 링크다.

```shell
$ kubectl exec -it fortune-configmap-volume -c web-server --Is -1A
/etc/nginx/conf.d
total 4
drw xr-xr-x . . . 12:15 . .4984_09_04_12_15_06.865837643
lrwxrwxrwx ... 12:15 ..data -> . .4984_09_04_12_15_06.865837643
lrwxrwxrwx ... 12:15 my-nginx-config.conf -> . .data/my-nginx-config.conf 
lrwxrwxrwx ... 12:15 sleep-interval -> . .data/sleep-interval
```

#### 이미 존재하는 디렉터리에 파일만 마운트 했을 때 없데이트가 되지 않는 것 이해하기

한가지 주의 사항은 컨피그맵 볼륨 업데이트와 관련이 있다. 만약 전체 볼륨 대신 단일 파일을 컨테이너에 마운트한 경우 파일이 업데이트 되지 않는다.
만약 개별 파일을 추가하고 원본 컨피그 맵을 업데이트 할 때 파일을 업데이트 해야 하는 경우 한 가지 해결 방법은 전체 볼륨을 다른 디렉터리에
마운트한 다음 다음 해당 파일을 가리키는 심볼릭 링크를 생성하는 것이다.

#### 컨피그맵 업데이트의 결과 이해하기

이후에는 변경된 configmap에 따라서 동작하는 것을 확인할 수 있다.

컨피그맵 업데이트의 결과 이해하기
컨테이너의 가장 중요한 기능은 불변성이다. 즉, 동일한 이미지에서 생성된 여러 실행 컨테이너 간에 차이가 없는지 확인할 수 있으므로 컨테이너를 실행하는 데 사용되는 컨피그맵을 수정해 이 불변성을 우회하는 것이 잘못된 것일까?

애플리케이션이 설정을 다시 읽는 기능을 지원하지 않는 경우에 심각한 문제가 발생한다. 이로 인해 서로 다른 설정을 가진 인스턴스가 실행되는 결과를 초래한다. 컨피그맵을
변경한 이후 생성된 파드는 새로운 설정을 사용하지만 예전 파드는 계속해서 예전 설정을 사용한다. 그리고 이것은 새로운 파드에만 국한되는 문제가 아니다. 파드 컨테이너가 어떠 한 이유로든 다시 시작되면 새로운 프로세스는 새로운 설정을 보게 된다. 따라서 애플리케 이션이 설정을 자동으로 다시 읽는 기능을 가지고 있지 않다면, 이미 존재하는 컨피그맵을 (파드가 사용하는 동안) 수정하는 것은 좋은 방법이 아니다.
애플리케이션이 다시 읽기를 지원한다면, 컨피그맵을 수정하는 것은 그리 큰 문제는 아니다. 하지만 컨피그맵 볼륨의 파일이 실행 중인 모든 인스턴스에 걸쳐 동기적으로 업데이트되지 않기 때문에, 개별 파드의 파일이 최대 1분 동안 동기화되지 않은 상태로 있을 수 있음을 알고 있어야 한다.
