# Section 7 - 볼륨에 대하여

## Volume

쿠버네티스 볼륨은 파드의 구성 요소로 컨테이너와 동일하게 파드 스펙에서 정의된다. 볼륨은 독립적인 쿠버네티스 오브젝트가 아니므로 자체적으로 생성, 삭제될 수 없다.
볼륨은 파드의 모든 컨테이너에서 사용 가능하지만 접근하려면 컨테이너에서 각각 마운트돼야 한다. 각 컨테이너에서 파일시스템의 어느 경로에나 불륨을 마운트할 수 있다.  

- .spec.volumes : 파드에 제공할 볼륨을 지정하고 
- .spec.containers[*].volumeMounts 의 컨테이너에 해당 볼륨을 마운트할 위치를 선언한다.  

컨테이너의 프로세스는 컨테이너 이미지의 최초 내용물과 컨테이너 안에 마운트된 볼륨으로 구성된 파일시스템을 보게된다.  
프로세스는 컨테이너 이미지의 최초 내용물에 해당되는 루트 파일 시스템을 보게 된다. 쓰기가 허용된 경우, 해당 파일 시스템에  
쓰기 작업을 하면 추후 파일 시스템에 접근할 때 변경된 내용을 보게 될 것 이다.    

볼륨은 이미지의 특정 경로에 마운트 된다. 파드에 정의된 각 컨테이너에 대해, 컨테이너가 사용할 각 볼륨을 어디에 마운트할지 명시해야 한다.  

### AWS EBS 구성 예시 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-ebs
spec:
  containers:
  - image: registry.k8s.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /test-ebs
      name: test-volume
  volumes:
  - name: test-volume
    # 이 AWS EBS 볼륨은 이미 존재해야 한다.
    awsElasticBlockStore:
      volumeID: "<volume-id>"
      fsType: ext4
```

### 사용 가능한 볼륨 유형 소개

- emptyDir : 일시적인 데이터를 저장하는데 사용되는 간단한 빈 디렉터리다.
- hostPath : 워커 노드의 파일 시스템을 파드의 디렉터리로 마운트 하는 데 사용한다.
- gitRepo : 깃 리포지터리의 컨텐츠를 체크아웃해 초기화한 볼륨이다.
- nfs : NFS 공유를 파드에 마운트 한다.
- gcePersistenceDisk, aswElasticBlockStore, azureDisk : 클라우드 제공자의 전용 스토리지를 마운트 하는데 사용한다.
- cinder, cephfs, iscsi, flocker, glusterfs, quobyte, rbd, flexVolume, vsphere Volume, photonPersistentDis, scaleIO : 다른 유형의 네트워크 스토리지를 마운트 하는데 사용한다.
- configMap, secret, downwardAPI : 쿠버네티스 리소스나 클러스터 정보를 파드에 노출하는데 사용되는 특별한 유형의 볼륨이다.
- persistentVolumeClaim: 사전에 혹은 동적으로 프로비저닝된 퍼시스턴스 스토리지를 사용하는 방법이다.

### 볼륨을 사용한 컨테이너 간 데이터 공유

### emptyDir 볼륨 사용

가장 간단한 불륨 유형은 emtpyDir 볼륨으로 어떻게 파드에 볼륨을 정의하는지 첫 번째 예제에서 살펴보자. 이름에서 알 수 있듯이 불륨이 빈 디렉터리로 시작된다. 파드에 실행 중인 애플리케이션은 어떤 파일이든
볼륨에 쓸 수 있다. 볼륨의 라이프 사이클이 파드에 묶여 있으므로 파드가 삭제되면 볼륨의 컨텐츠는 사라진다.

emptyDir 볼륨은 동일 파드에서 실행 중인 컨테이너 간 파일을 공유할 때 유용하다. 그러나 단일 컨테이너에서도 가용한 메모리에 넣기에 큰 데이터 세트의 정렬 작업을 수행하는 것과 같이
임시 데이터를 디스크에 쓰는 목적인 경우 사용할 수 있다.

#### Sample

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: registry.k8s.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
    ports:
    - containerPort: 80 
      protocol: TCP 
  volumes:
  - name: cache-volume
    emptyDir:
      sizeLimit: 500Mi
```

실행중인 파드를 통해서 점검하기

```shell
$ k port-forward fortune test-pd
```

#### 깃 리포지터리를 볼륨으로 사용하기 


## 워커 노드 파일 시스템의 파일 접근

대부분의 파드는 호스트 노드를 인식하지 못하므로 노드의 파일 시스템에 있는 어떤 파일에도 접근하면 안된다. 그러나
특정 시스템 레벨의 파드는 노드의 파일을 읽거나 파일 시스템을 통해 노드 디바이스에 접근하기 위해 노드의 파일시스템을
사용해야 한다.

### hostPath

![](https://keepinmindsh.github.io/lines/assets/img/k8s-hostpath_structure.png)

- hostPath 볼륨의 컨텐츠는 삭제되지 않는다.
    - 파드가 삭제되면 다음 파드가 호스트의 동일 경로를 가리키는 hostPath 볼륨을 사용하고, 이전 파드와 동일한 노드에 스케줄링된다는 조건에서 이전 파드가 남긴 모든 항목을 볼 수 있다.
    - hostPath 볼륨은 파드가 어떤 노드에 스케쥴링 되느냐에 따라 민감하기 때문에 일반적인 파드에 사용하는 것은 좋은 생각이 아니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: registry.k8s.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /test-pd
      name: test-volume
  volumes:
  - name: test-volume
    hostPath:
      # directory location on host
      path: /data
      # this field is optional
      type: Directory
```

> 만약 특정 폴더 및 파일을 생성하고 싶은 경우, 아래와 같이 작성할 수 있는 데,  
> 중요한 부분은 만약 지정한 폴더 경로의 상위 폴더가 존재하지 않으면 만들어주지 않으며 해당 파드는 기동중에 에러가 발생할 수 있다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-webserver
spec:
  containers:
  - name: test-webserver
    image: registry.k8s.io/test-webserver:latest
    volumeMounts:
    - mountPath: /var/local/aaa
      name: mydir
    - mountPath: /var/local/aaa/1.txt
      name: myfile
  volumes:
  - name: mydir
    hostPath:
      # Ensure the file directory is created.
      path: /var/local/aaa
      type: DirectoryOrCreate
  - name: myfile
    hostPath:
      path: /var/local/aaa/1.txt
      type: FileOrCreate
```

**노드의 시스템 파일에 읽기/쓰기를 하는 경우에만 hostPath 볼륨을 사용한다는 것을 기억하라. 여러 파드에 걸쳐 데이터를 유지하기 위해서는 절대 사용하지 말라**

## 퍼시스턴트 스토리지 사용

파드에서 실행 중인 애플리케이션이 디스크에 데이터를 유지해야하고 파드가 다른 노드로 재스케쥴링 된 경우에도 동일한 데이터를
사용해야 한다면 지금까지 언급한 볼륨 유형은 사용할 수 없다. 이러한 데이터는 어떤 클러스터 노드에서 접근이 필요하기 때문에
NAS 유형에 저장돼야 한다.

### GCE 퍼시스턴트 디스크를 파드 볼륨으로 사용하기

![](https://keepinmindsh.github.io/lines/assets/img/k8s-gcepersistencedisk.png)

동일 location 에 퍼시스턴트 디스크를 만들기 위해서 gke clusters의 생성 위치를 확인한다.

```shell
$ gcloud container clusters list 
NAME           LOCATION       MASTER_VERSION   MASTER_IP      MACHINE_TYPE  NODE_VERSION     NUM_NODES  STATUS
lines-cluster  us-central1-c  1.24.9-gke.3200  104.197.11.14  e2-medium     1.24.9-gke.3200  3          RUNNING

```

>[GCE Persistence Volume 사용하기](https://cloud.google.com/compute/docs/disks/add-persistent-disk#gcloud)

#### GCE 퍼시스턴트 디스크 생성하기

- 퍼시스턴트 디스크 생성하기

```shell
$ gcloud compute disks create --size=10GB --zone=us-central1-c mongodb 
NAME     ZONE           SIZE_GB  TYPE         STATUS
mongodb  us-central1-c  10       pd-standard  READY

```

- 생성된 퍼시스턴트 디스크에 대해서 파드 생성하기

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mongodb
  labels:
    name: mongodb
spec:
  volumes:
    - name: mongodb-data
      gcePersistentDisk:
        pdName: mongodb
        fsType: ext4
  containers:
  - name: mongodb
    image: mongo
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
    ports:
      - containerPort: 8080
        protocol: TCP
    volumeMounts:
      - mountPath: /data/db
        name: mongodb-data
```

```shell
$ k create -f 007_kuberntes_in_action/p277_pod_with_gce_persistence_disk/mongodb-pod-gcepd.yaml
pod/mongodb created
```

- 생성된 DISK 삭제하기

```shell
$ gcloud compute disks delete my-disk --zone=us-east1-a
```

```shell
gcloud compute disks delete mongodb --zone=us-central1-c
The following disks will be deleted:
 - [mongodb] in [us-central1-c]

Do you want to continue (Y/n)?  Y
Deleted [https://www.googleapis.com/compute/v1/projects/lines-infra/zones/us-central1-c/disks/mongodb].
```

### 다른 유형의 볼륨 사용하기

GCE 퍼시스턴트 디스크 볼륨은 쿠버네티스 클러스터를 구글 쿠버네티스 엔진에서 실행 중이기 때문이었다. 다른 곳에서 실행 중이라면 기반 인프라 스트럭처에 따라
다른 유형의 볼륨을 사용해야 한다.

- AWS Elastic Block Store 볼륨 사용하기

```yaml
apiVersion: v1 
kind: Pod 
metadata: 
  name: mongodb 
spec: 
  volumes: 
    - name: mongodb-data 
      awsElasticBlockStore: 
        volumeId: my-volume 
        fsType: ext4 
  containers: 
    - ... 
```

- nfs 볼륨을 사용하는 파드

```yaml
volumes: 
  - name: mongodb-data 
    nfs: 
      server: 1.2.3.4 
      path: /some/path 
```

## 기반 스토리지 기술과 파드 분리

지금까지 살펴본 모든 퍼시스턴트 볼륨 유형은 파드 개발자가 실제 네트워크 스토리지 인프라스트럭처에 관한 지식을 갖추고 있어야 한다.
NFS 기반의 볼륨을 생성하려면 개발자는 NFS 익스포트가 위치하는 실제서버를 알아야한다.
이는 인프라스트럭처의 세부사항에 대한 걱정을 없애고, 클라우드 공급자나 온프레미스 데이터센터를 걸쳐 이식 가능한 애플리케이션을 만들고,
애플리케이션과 개발자로부터 실제 인프라스트럭처를 숨긴다는 쿠버네티스의 기본 아이디어에 반한다.

### 퍼시스턴트 볼륨과 퍼시스턴트 볼륨 클레임

인프라스트럭처의 세부 사항을 처리하지 않고 애플리케이션이 쿠버네티스 클러스터에 스토리지를 요청할 수 있도록 하기위해 새로운 리소스 두 개가 도입됐다.

- 퍼시스턴트 볼륨
- 퍼시스턴트 볼륨 클레임

> [Use persistent disks with multiple readers](https://cloud.google.com/kubernetes-engine/docs/how-to/persistent-volumes/readonlymany-disks)

개발자가 파드에 기술적인 세부 사항을 기재한 볼륨을 추가하는 대신 클러스터 관리자가 기반 스토리지를 설정하고 쿠버네티스 API 서버로 퍼시스턴트 불륨 리소스를 생성해
쿠버네티스에 등록한다. 퍼시스턴트 볼륨이 생성되면 관리자는 크기와 지원 가능한 접근 모드를 지정한다.

클러스터 사용자가 파드에 퍼시스턴트 스토리지를 사용해야 하면 먼저 최소 크기와 필요한 접근 모드를 명시한 퍼시스턴트볼륨클레임 매니페스트를 생성한다.  
그런 다음 사용자는 퍼시스턴트볼륨클레임 매니페스트를 쿠버네티스 API 서버에 게시하고 쿠버네티스는 적절한 퍼시스턴트볼륨을 찾아 클레임에 볼륨을 바인딩한다.

#### 퍼시스턴트 볼륨

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0003
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: slow
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /tmp
    server: 172.17.0.2
```

#### 퍼시스턴트 볼륨 클레임

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: example-vol-default
provisioner: vendor-name.example/magicstorage
parameters:
  resturl: "http://192.168.10.100:8080"
  restuser: ""
  secretNamespace: ""
  secretName: ""
allowVolumeExpansion: true
```

## 퍼시스턴트 볼륨의 동적 프로비저닝

### 컨피그 맵의 활용 이유를 위한 사전 분석

## 요약

- 다중 컨테이너 파드 생성과 파드의 컨테이너들이 불륨을 파드에 추가하고 각 컨테이너에 마운트해 동일한 파일로 동작하게 한다.
- emptyDir 볼륨을 사용해 임시, 비영구 데이터를 저장한다.
- gitRepo 볼륨을 사용해 파드의 시작 지점에 깃 리포지터리의 콘텐츠로 디렉터리를 쉽게 채운다.
- hostPath 볼륨을 사용해 호스트 노드의 파일에 접근한다.
- 외부 스토리지를 볼륨에 마운트해 파드가 재시작돼도 파드의 데이터를 유지한다.
- 퍼시스턴트 볼륨과 퍼시스턴트 볼륨 클레임을 사용해 파드와 스토리지 인프라 스트럭처를 분리한다.
- 각 퍼시스턴트 볼륨 클레임을 위해 퍼시스턴트 볼륨을 원하는 스토리지 클래스로 동적 프로비저닝 한다.
- 퍼시스턴트 볼륨 클레임을 미리 프로비저닝된 퍼시스턴트 볼륨과 바인딩하고자 할 때 동적 프로지저너가 간섭하는 것을 막는다. 