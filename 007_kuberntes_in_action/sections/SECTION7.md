# Section 7 - 볼륨에 대하여

## Volume

쿠버네티스 볼륨은 파드의 구성 요소로 컨테이너와 동일하게 파드 스펙에서 정의된다. 볼륨은 독립적인 쿠버네티스 오브젝트가 아니므로 자체적으로 생성, 삭제될 수 없다.
볼륨은 파드의 모든 컨테이너에서 사용 가능하지만 접근하려면 컨테이너에서 각각 마운트돼야 한다. 각 컨테이너에서 파일시스템의 어느 경로에나 불륨을 마운트할 수 있다.

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