# Section 20 - 애플리케이션 개발을 위한 모범 사례 

## 모든 것을 하나로 모아보기

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_usage_001.png) 

일반적인 애플리케이션 매니페스트에는 하나 이상의 디플로이먼트나 스테이트풀셋 오브젝트가 포함된다. 여기에는 하나 이상의 컨테이너 포함된 파드 템플릿, 각 컨테이너에  
대한 라이브니스 프로브와 컨테이너가 제공하는 서비스에 대한 레디니스 프로브가 포함된다. 다른 사람에게 서비스를 제공하는 파드는 하나 이상의 서비스로 노출된다.  
클러스터 외부로 통신할 수 있어야 하는 경우 서비스는 로드밸런서나 노드포트 유형 서비스로 구성되거나 인그레스 리소스로 노출된다.   

  애플리케이션에는 환경 변수를 초기화하거나 파드에 컨피그맵 볼륨으로 마운트 되는 하나 이상의 컨피그 맵이 포함돼 있다. 특정 파드는 emptyDir 또는 gitRepo 볼륨과  
같은 추가 볼륨을 사용하는 반면, 퍼시스턴트 스토리지가 있어야 하는 파드는 퍼시스턴트 볼륨 클레임 볼륨을 사용한다. 퍼시스턴트 볼륨 클레임은 애플리케이션 매니페스트에 속하며,  
이에 의해 참조된 스토리지클래스는 시스템 관리자가 미리 생성된다.   

 경우에 따라 애플리케이션에서 잡이나 크롭잡을 사용해야 한다. 데몬셋은 일반적으로 애플리케이션 디플로이먼트 일부는 아니지만 일반적으로 시스템 운영자가 노드 전체 또는 일부에  
시스템 서비스를 실행하려고 생성한다. 수평 파드 오토 스케일러는 개발자가 매니페스트에 포함시키거나 나중에 운영 팀이 시스템에 추가한다. 또한 클러스터 관리자는 
제한 범위와 리소스 쿼터 오브젝트를 생성해 개별 파드와 모든 파드의 리소스 사용량을 제어할 수 있다.  

애플리케이션이 배포되면 다양한 쿠버네티스 컨트롤러에 의해 오브젝트가 추가적으로 자동 생성된다. 여기에는 엔드포인트 컨트롤러로 생성된 서비스 엔드포인트 오브젝트, 디플로이먼트 컨트롤러로 생성된  
레플리카셋과 레플리카셋, 잡, 크론잡, 스테이트풀셋, 데몬셋 컨트롤러로 생성된 실제 파드가 포함된다. 

리소스는 종종 체계적으로 유지되기 위해서 하나 이상의 레이블을 지정한다. 이는 파드에만 적용되는것이 아니라 다른 모든 리소스에도 적용된다. 이는 파드에만 적용되는 것이 아니라 다른 모든 리소스에도 적용된다.  
레이블 외에도 대부분의 리소스에는 각 리소스를 설명하거나 해당 담당자 또는 팀의 연락처 정보를 나열하거나 관리와 기타 도구에 관한 추가 메타 데이터를 제공하는 어노테이션이 포함돼 있다. 

## 파드 라이프 사이클 이해

### 애플리케이션을 종료하고 파드 재배치 예상하기 

#### 로컬 IP와 호스트 이름 변경 예상하기  

파드가 종료되고 다른 곳에서 실행되면 새로운 IP 주소뿐만 아니라 새로운 이름과 호스트 이름을 갖는다. 대부분의 스테이트리스 애플리케이션은 일반적으로 문제 없이  
처리할 수 있지만 스테이트풀 애플리케이션은 그렇지 않다. 스테이트 풀 애플리케이션을 스테이트풀셋으로 실행할 수 있다는 사실을 알고 있으므로 스케줄링을 조정한 후 새 노드에서  
애플리케이션을 시작할 때도 여전히 이전과 동일한 호스트 이름과 퍼시스턴트 상태를 볼 수 있다. 그럼에도 파드의 IP는 변경될 것이다. 이를 위해서 애플리케이션이 미리 준비되어 잇어야 한다. 
따라서 애플리케이션 개발자는 클러스터된 애플리케이션의 구성원을 IP 주소 기반으로 하면 안 되며 호스트 이름을 기반으로 할 때에는 항상 스테이트 풀셋을 사용해야 한다. 

#### 디스크에 기록된 데이터가 사라지는 경우 예상하기 

애플리케이션이 디스크에 데이터를 쓰는 경우 애플리케이션이 쓰는 위치에 퍼시스턴트 스토리지를 마운트 하지 않으면 애플리케이션이 새 파드로 시작된 후에  
해당 데이터를 사용하지 못할 수도 있다는 것! 

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_usage_002.png)

#### 컨테이너를 다시 사용하더라도 데이터를 보존하기 위해 볼륨 사용하기 

데이터가 손실되지 않도록 하려면 최소한 파드 범위의 볼륨을 사용해야 한다. 볼륨이 파드와 함께 시작하고 종료하기 때문에 새 컨테이너는 이전 컨테이너가 볼륨에 
기록한 데이터를 재사용 할 수 있다. 

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_usage_003.png)

컨테이너를 재시작할 대 파일을 보존하려고 불륨을 사용하는 것은 좋은 생각이지만 항상 그런 것은 아니다. 데이터가 손상돼 새로 생성된 프로세스가 다시 크래시되면 
어떻게 할까? 이로 인해 연속 크래시 루프가 발생한다. 볼륨을 사용하지 않은 경우 새컨테이너가 처음부터 시작돼 크래시하지 않을 가능성이 높다. 
컨테이너를 재시작할 때 파일을 보존하려고 불륨을 사용하는 것은 양날의 검이다. 

#### 종료된 파드 또는 부분적으로 종료된 파드를 다시 스케쥴링 하기 

파드의 컨테이너가 계속 크래시 되면 Kubelet은 계속 파드를 재시작한다. 재시작 간격은 5분이 될 때까지 간격이 증가한다. 이 5분 간격 동안 컨테이너의 프로세스가  
실행되고 있지 않기 때문에 파드는 기본적으로 종료된다. 멀티 컨테이너 파드인 경우 엄밀하게 말해 특정 컨테이너가 정상적으로 작동할 수 있으므로 파드는 일부만 종료된 것으로  
볼 수 있다. 그러나 파드에 컨테이너가 하나만 포함돼 있으면 더 이상 프로세스가 실행되지 않기 때문에 파드가 실제로 종료된 것이라 완전히 쓸모 없어진다.  

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_usage_004.png)

파드가 삭제되고 다른 노드에서 성공적으로 실행될 수 있는 다른 파드 인스턴스로 교체되기 원할 것이다. 결국 다른 노드에서 나타나지 않은 노드 관련 문제로 인해 컨테이너  
가 중단될 수 있다. 슬프게도, 그렇지 않다. 레플리카셋 컨트롤러는 파드가 죽은 상태가 됐는지 상관하지 않는다. 관심 있는 것은 파드 수가 의도하는 레플리카 수와 일츠하느냐 하는 것이다. 

```shell
kubectl get po 
# 파드 상태는 컨테이너가 계속 크래시되기 때문에 Kubelet 재시작을 지연한다고 나타난다. 

kubectl describe rs crashing-pods 
# 현재 레플리카와 의도하는 레플리카가 일치하기 때문에 컨트롤러가 아무런 조치도 취하지 않는다. 
# 레플리카 세 개가 실행 중이다. 

kubectl describe po crashing-pods-fitcd # kubectl 은 파드의 상태가 실행 중임을 보여준다.   
```
### 원하는 순서로 파드 시작 

파드에서 실행되는 애플리케이션과 수동으로 관리되는 애플리케이션의 또 다른 차이점은 애플리케이션을 배포하는 담당자가 애플리케이션 간의 의존성을 알고 있다는 것이다.  
이것은 애플리케이션을 순서대로 시작할 수 있게 해준다. 

#### 파드 시작 방법 

쿠버네티스 API 서버는 YAML/JSON의 오브젝트를 나열된 순서대로 처리하지만 이는 etcd에 순서대로 기록됨을 의미한다. 파드가 그 순서대로 시작된다는 보장은 없다. 

#### 초기화 컨테이너 소개 

파드는 여러 개의 초기화 컨테이너를 가질 수 있다. 순차적으로 실행되며 마지막 컨테이너가 완료된 후에 파드의 주 컨테이너가 시작된다. 즉, 초기화 컨테이너를 사용해  
파드의 주 컨테이너 시작을 지연시킬 수 있다. 초기화 컨테이너가 시작되면, 초기화 컨테이너는 종료되고 주 컨테이너가 시작될 수 있게 한다. 이렇게 하면 주 컨테이너가 준비되기 
전까지 서비스를 사용하지 않게 된다. 

#### 파드에 초기화 컨테이너 추가 

초기화 컨테이너는 주 컨테이너와 같이 파드 스펙에 정의될 수 있지만 spec.initContainers 필드에 정의할 수도 있다.  

```yaml
spec: 
  initContainers:
  - name: init 
    image: busybox 
    command: 
    - sh
    - -c 
    - 'while true; do echo "Waiting for fortune service to come up..."; 
       wget http://fortune -q -T 1 -O /dev/null > /dev/null 2> /dev/nul; 
       && break; sleep 1; done; echo "Service is up! Starting main container."'
```

```shell
# 위의 파드를 배포하면 초기화 컨테이너 중 0개가 완료됐음을 나타낸다. 
$ kubectl get po 

# STATUS 열은 하나의 초기화 컨테이너 중 0개가 완료됐음을 나타낸다. kubectl logs 로 초기화 컨테이너의 로그를 볼 수 있다. 
$ kubectl logs fortune-client -c init 
```

#### 파드간 의존성 처리를 위한 모범 사례 

전제 조건이 충족될 때까지 파드 컨테이너의 주 컨테이너의 시작을 지연시키는 데 초기화 컨테이너를 사용하는 방법을 살펴봤다.   
애플리케이션이 시작되기 전 준비 상태가 되기 위해 의존해야할 서비스를 필요로 하지 않도록 애플리케이션을 만드는 것이 좋은 방법이다.  
결국 애플리케이션이 실행 상태가 되더라도 서비스는 나중에 오프라인이 될 수 있다. 

### 라이프사이클 훅 추가 

파드는 라이프사이클 훅 두가지를 정의할 수 있다. 

- 시작 후 훅 
- 종료 전 훅

#### 컨테이너의 시작 후 라이프사이클 훅 사용  

시작 후 훅은 컨테이너의 주 프로세스가 시작된 직후 시작된다. 애플리케이션이 시작될 대 추가 작업을 수행하는 데 사용한다. 
물론 컨테이너에서 실행 중인 애플리케이션 개발자라면 항상 애플리케이션 코드 내에서 해당 작업을 수행할 수 있다.   
그러나 다른 사람이 개발한 애플리케이션을 실행할 때는 대부분을 소스코드를 수정하고 싶어 하지 않는다. 시작 훅을 사용하면  
애플리케이션을 건드리지 ㅇ낳고도 추가 명령을 실행할 수 있다. 애플리케이션이 시작되고 있는 외부 리스너에게 시그널을 보내거나 
애플리케이션을 초기화하는 작업을 시작할 수 있다.  

```yaml
apiVersion: v1 
kind: Pod 
metadata: 
  name: pod-with-poststart-hook
spec: 
  containers:
  - image: luksa/kubia 
    name: kubia 
    lifecycle: 
      posStart: 
        exec: 
          command: 
          - sh 
          - -c 
          - "echo 'hook will fail with exit code 15'; sleep 5; exit 15"
```

명령어 기반이 시작 후 훅의 표준 출력과 표준 오류 출력은 어디에도 로깅되지 않으므로 훅 호출 프로세스를 컨테이너의 파일 시스템에 있는 파일에 기록하고 다음과 같이 파일의 내용을 확인할 수 있습니다 

```shell
$ kubectl exec my-pod cat logfile.txt 
```

#### 컨테이너 종료 전 라이프사이클 훅 사용 

컨테이너가 종료되기 직전에 종료전 훅이 실행된다. 컨테이너를 종료해야할 경우 kubelet은 종료전 혹을 실행한 다음 SIGTERM을 전송한다.  
컨테이너가 SIGTERM 신호를 수신해도 정상적으로 종료되지 않으면 종료 전 훅을 사용해 컨테이너를 정상적으로 종료할 수 있다.  
또한 애플리케이션 내에서 이런 작업을 구현하지 않고도 종료 전 임의의 작업을 수행하도록 하는데 사용할 수 있다. 

```yaml 
lifecycle: 
  preStop: 
    httpGet: # HTTP GET 요청을 수행하는 종료 전 훅  
      port: 8080 # http://POD_IP:8080/shutdown 으로 요청을 전송한다. 
      path: shutdown 
```

정의된 종료 전 훅은 kubelet이 컨테이너 종료를 시작하자마자 http://POD_IP:8080/shutdown 에 HTTP GET 요청을 보낸다.  
예제에 표시된 포트와 경로 외에도 필드 스키마 와 호스트 뿐만이 아니라 요청에 전송해야 하는 http 헤더도 설정할 수 있다.  
호스트 필드의 기본 값은 파드 IP다. localhost는 파드가 아닌 노드를 참조하므로 localhost로 설정하면 안 된다.   
 시작 후 훅과 달리 컨테이너는 훅 결과에 상관없이 종료된다. 명령 기반 훅을 사용할 때 HTTP 오류 응답 코드 또는 0이 아닌  
종료 코드는 컨테이너 종료를 막지 않는다. 종료전 혹은 실패하면 파드의 이벤트 중에 FailedPreStopHook 경고 이벤트가 표시되지만  
곧 파드가 삭제되기 때문에 종료 전 훅이 제대로 작동하지 않았다는 것을 알지 못할 수도 있다.  

### 파드 셧다웃 이해하기 

파드의 종료는 API 서버로 파드 오브젝트를 삭제하면 시작된다.  
HTTP DELETE 요청을 수신하면 API 서버는 아직 오브젝트를 삭제하지 ㅇ낳고 그 안에 deleteTimeStamp 필드만 설정한다.  
그리고 deletedTimestamp 필드가 설정된 파드가 종료된다.  

Kubelet은 파드를 종료해야 함을 확인하면 각 파드의 컨테이너를 종료하기 시작한다.  
각 컨테이너에 정상적으로 종료하는 데 시간이 걸리지만 시간이 제한돼 있다. 이 시간을 종료 유에 기간이라고 하며 파드별로 구성할 수 있다.  
종료 프로세스가 시작되자마자 타이머가 시작된다. 

- 종료전 훅을 실행하고 완료될 때까지 기다린다. 
- SIGTERM 신호를 컨테이너의 주 프로세스로 보낸다. 
- 컨테이너가 완전히 종료될 때 까지 또는 종료 유예 이간이 끝날때 까지 기다린다. 
- 아직 정상적으로 종료되지 않은 경우 SIGKILL로 프로세스를 강제 종료한다. 

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_usage_005.png) 

#### 종료 유예 기간 한정 

종료 유예 기간은 포스 스펙에서 terminateGracePeriodSeconds 필드로 설정할 수 있다.  
기본 값은 30이고 파드의 컨테이너는 강제 종료되기 전에 정상 종료할 수 있도록 30초가 주어진다. 

> 프로세스가 시간 내에 종료될 수 있도록 유예 기간을 충분히 길게 설정해야 한다.  

```shell
$ kubectl delete po mypod --grace-period=5  

$ kubectl delete po mypod --grace-period=0 --force 
```

이 옵션을 사용할 때는 특히 스테이트 풀셋 파드에 주의한다. 스테이트풀셋 컨트롤러는 동일한 파드의 두 인스턴스를  
동시에 실행하지 않도록 유의한다. 파드를 강제 삭제하면 삭제된 파드의 컨테이너가 종료될 때 까지 기다리지 않고 컨트롤러가 
교체 파드를 생성하게 된다. 즉, 동일한 파드의 두 인스턴스가 동시에 실행돼 스테이트 풀 클러스터가 오작동 할 수 있다.  
파드가 더이상 실행되고 있지 않거나 클러스터의 다른 멤버와 대화할 수 없는 경우 스테이트풀 파드를 강제로 삭제한다.

#### 애플리케이션에서 적절한 셧다운 핸들러 구현 

- 컨테이너가 종료해도 파드 전체가 종료되는 것은 아니다. 
- 프로세스가 종료되기 전에 종료 절차가 끝난 것이라는 보장이 없다. 

애플리케이션의 정상적인 종료가 수행되기 전에 종료 유예 기간이 만료될 뿐만 아니라 컨테이너의 셧다운 단계 중간에  
파드를 실행하는 노드가 실패하는 경우에도 발생한다. 그 후, 노드가 다시 시작하는 경우에도 kubelet은 셧다운 절차를  
다시 시작하지 않는다. 파드가 전체 셧다운 단계 전체를 완료할 수 있다는 보장은 전혀없다.  

#### 전용 셧다운 절차 파드를 사용해 중요한 셧다운 절차 대체  

한 가지 해결책은 애플리케이션이 새 파드를 실행하는 새로운 잡 리소스를 만드는 것이며, 그 역할은 삭제된 파드의 데이터를 나머지 파드로 옮기는 것이다.  
그러나 주의해야 할 것은 애플리케이션이 잡 오브젝트를 매번 만들 수 있다는 보장이 없다는 것이다.  

이 문제를 처리하는 적절한 방법은 분산된 데이터의 존재를 확인하는 전용 파드를 항상 실행하는 것이다. 이 파드가 분산된 데이터를 찾아 나머지 파드로  
마이그레이션할 수 있다.  

스테이트 풀셋을 스케일 다운하면 퍼시스턴트볼륨클레임은 혼자 남고, 묶인 퍼시스턴트볼륨 안에 있는 데이터는 그대로 남아 있게 된다.  
스케일 업할 때 퍼시스턴트 볼륨이 새로운 파드 인스턴스에 다시 연결되지만 스케일업이 발생하지 않는 경우는 어떻게 될까?  
따라서 스테이트 풀셋을 사용하는 경우에도 데이터 마이그레이션 파드를 실행하는 것이 좋다. 애플리케이션 업그레이드 중에 마이그레이션이  
발생하지 않도록, 마이크레이션을 수행하기 전에 스테이트 풀셋 파드가 다시 시작되도록 시간을 줘 데이터 마이그레이션 파드가 잠시 대기하도록  
구성할 수 있다.

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_usage_006.png)

## 모든 클라이언트 요청의 적절한 처리 보장

### 파드가 시작될 때 클라이너트 연결 끊기 방지 

애플리케이션이 들어오는 요청을 올바르게 처리할 준비가 됐을 때만 레디니스 프로브가 성공을 반환하도록 해야 한다.  
첫 번째 단계는 HTTP GET 레디니스 프로브를 추갛고 애플리케이션의 기본 URL을 가리키는 것이다. 대부분 정상 동작하여  
애플리케이션에서 특별한 레디니스 엔드 포인트 구현하지 않아도 된다.  


#### 파드 삭제시 발생하는 이벤트의 순서 이해 

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_usage_007.png)

A 이벤트 순서에서 kubelet이 파드를 종료해야 한다는 알림을 받으면 설명한 대로 셧다운 순서를 시작한다.   
애플리케이션이 클라이언트 요청을 즉지 받지 ㅇ낳음으로써 SIGTERM에 응답하는 경우 애플리케이션에 연결하려는 모든 클라이언트는 
connection refused 오류를 수신한다. API 서버에서 kubelet까지 바로 파드가 삭제되는 시점까지 이 작업이 수행되는 데 걸리는 시간은 
비교적 짧다.   

이제 다른 일련의 이벤트가 발생하는 상황을 살펴보자. 쿠버네티스 컨트롤 플레인의 컨트롤러 매니저가 실행되는 엔드포인트 컨트롤러는 파드가  
삭제됐다는 알림을 받으면 파드가 속한 모든 서비스의 엔드 포인트에서 파드를 제거한다. API 서버에서 REST 요청을 보내 엔드포인트 API 오브젝트를  
수정해 이를 수행한다. 그런 다음 API 서버는 엔드 포인트 오브젝트를 보고 있는 모든 클라이언트에게 알린다.  

이러한 감시자 중에는 워커 노드에서 실행되는 kube-proxy가 있다. 그런 다음 각 프록시는 노드에서 iptables 규칙을 업데이트 해 새 연결이 종료되는 
파드로 전달되지 않도록 한다. 여기서 iptables 규칙을 제거해도 기존 연결에는 영향을 미치지 않는다. 파드에 이미 연결된 클라이언트는 기존 연결로 파드에  
추가 요청을 계속 보낸다.   

이 두 사건의 이벤트는 동시에 발생한다. 파드에서 애플리케이션의 프로세스를 종료하는 데 걸리는 시간은 iptables 규칙을 업데이트하는 데 필요한 시간보다  
약간 짧다. 이벤트가 먼저 엔드 포인트 컨트롤러에 도달한 후 API 서버 및 AI에 새 요청을 보내야 하므로 iptables 규칙을 업데이트하는 이벤트 체인이  
상당히 길어진다. 알림을 보고 프록시가 최종적으로 iptables 규칙을 수정하기 전에 API 서버가 kube-proxy로 알려야 한다. iptables 규칙이 모든 노드에서  
업데이트 되기 전에 SIGTERM 신호가 전송될 가능성이 높다. 

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_usage_008.png)

결과 적으로 파드는 종료 신호를 보낸 후에도 클라이언트 요청을 받을 수 있다. 애플리케이션이 서버 소켓을 닫고 연결 수락을 즉시 중지하면  
클라이언트가 "Connection Refused" 오류를 수신하게 된다.  

#### 안정적으로 애플리케이션을 종료하려면, 

- 몇 초를 기다린 후, 새 연결을 받는 것을 막는다. 
- 요청 중이 아닌 모든 연결 유지 연결을 닫는다. 
- 모든 활성 요청이 완료될 때까지 기다린다. 
- 그런 다음 완전히 셧다운한다.

![](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/k8s_usage_009.png)

최소한 몇 초 동안 대기하는 종료 전 훅을 추가하는 것이 좋다. 

```yaml
lifecycle: 
  preStop: 
    exec: 
      command: 
      - sh
      - -c 
      - "sleep 5"
```

## 쿠버네티스에서 애플리케이션을 쉽게 실행하고 관리할 수 있게 만들기

### 관리 가능한 컨테이너 이미지 만들기  


새 파드를 배포하고 확장하는 것은 빨라야 한다. 이때 불필요한 것이 없는 작은 이미지가 필요하다.  
Go 언어를 사용해 애플리케이션을 제작하는 경우 이미지에는 애플리케이션의 바이너리 실행 파일 외에 다른 것을
포함할 필요가 없다. 따라서 Go 기반 컨테이너 이미지는 작으므로 쿠버네티스에 매우 적합하다.

> 이런 이미지는 Dockerfile에서 FROM stratch 지시문을 사용한다.

### 이미지에 적절한 태그를 지정하고 imagePullPolicy를 현명하게 사용  

파드 매니페스트에서 latest 태그를 참조하면 개발 파드 레플리카가 실행 중인 이미지 버전을 알수 없기 때문에 문제가 발생할 수 있음을 알게 될 것이다. 
처음에 모든 파드 레플리카가 동일한 이미지 버전을 실행하더라도 latest 태그 아래에 이미지의 새 버전을 푸시하면 파드의 스케줄링을 조정하거나  
디플로이먼트를 확장하면 새 파드가 새 버전을 실행한다. 반면 이전 컨테이너는 여전히 이전 버전을 실행한다. 또한 latest 태그를 사용하면  
이전 버전의 이미지를 다시 푸시하지 않는 한 이전 버전으로 롤백할 수 없다.  
  개발을 제외하고 latest 버전 대신 적절한 버전 지정자를 포함하는 태그를 사용하는 것이 필수다. 변경 가능한 태그를 사용하는 경우 파드 스펙의 
imagePullPolicy 필드를 always로 설정해야 한다. 그러나 프로덕션 파드에서 이를 사용하면 큰 문제가 발생할 수 있다. imagePullPolicy가 always로  
설정되면 새 파드가 배포될 때마다 커테이너 런타임이 이미지 레지스트리에 접속해서 가져온다. 노드가 이미지가 수정됐는지 확인해야 하기 대문에 파드 시작 속도가  
약간 느려진다. 게다가 더 나쁜점은 이 정책은 레지스트리에 연결할 수 없는 경우 파드가 시작되지 않게 한다

### 일차원 레이블 대신 다차원 레이블 사용 

파드 뿐만 아니라 모든 리소스에 레이블을 지정하는 것을 잊지 말자. 각 리소스에 여러 레이블을 추가하면 각 개별 차원에서 레이블을 선택할 수 있다.  
이렇게 관리한 경우 리소스 수가 증가하면 운영팀은 감사하게 생각할 수 있다. 

- 리소스가 있는 애플리케이션 
- 애플리케이션 계층 
- 환경 
- 버전 
- 릴리스 유형 
- 테넌트 
- 샤스된 시스템용 샤드 

### 어노테이션으로 각 리소스 설명 

리소스에 정보를 추가하려면 어노테이션을 사용하자. 최소한 리소스에는 리소스를 설명하는 어노테이션과 담ㄷ아자의 연락처 정보가 포함된   
어노테이션이 포함돼야 한다.  

### 프로세스가 종료된 원인에 대한 정보 제공

특히 최악의 순간이 발생했다면 컨테이너가 왜 종료했는지 알아내야 하는 것보다 더 좌절스러운 것은없다. 로그 파일에 필요한 모든 디버그 정보를  
포함시켜 운영 담당자의 삶을 행복하고 편안하게 만들어주자.  
 그러나 문제 파악을 서욱 쉽게하고자 다른 쿠버네티스 기능을 사용해 컨테이너가 파드 상태에서 종료된 이유를 보여줄 수 있다.  
프로세스가 컨테이너의 파일시스템에 있는 특정 파일에 종료된 이유를 보여줄 수 있다. 프로세스가 컨테이너의 파일 시스템에 있는 특정  
파일에 종료 메시지를 작성하도록 할 수 있다. 이 파일의 내용은 컨테이너가 종료될 때 kubelet에서 읽고 kubectl describe pod의  
출력에 표시된다. 애플리케이션이 이 메커니즘을 사용하면 운영자는 컨테이너 로그를 보지 않아도 애플리케이션이 종료된 이유를 신속하게  
확인할 수 있다.  

```yaml
apiVersion: v1 
kind: Pod 
metadata: 
  name: pod-with-termination-message 
spec: 
  containers: 
  - image: busybox 
    name: main 
  terminationMessagePath: /var/termination-reason # 종료 메시지 파일의 기본 경로를 재정의 한다. 
  command: 
  - sh 
  - -c 
  - 'echo "I''ve had enough" > /var/termination-reason ; exit 1' # 컨테이너는 종료 직건에 파일에 메시지를 쓸 것이다. 
```

```shell
$ kubectl describe po 
```

### 애플리케이션 로깅 처리 

애플리케이션 로깅을 다루는 동안 애플리케이션이 파일 대신 표준 출력에 작성해야 한다는 점을 상기한다. 

kubectl logs 명령어로 로그를 쉽게 볼 수 있다. 

> 컨테이너가 크래시되고 새 컨테이너로 교체하면 새 컨테이너의 로그가 표시된다. 이전 컨테이너의 로그를 보려면  
> kubectl logs 와 함께 --previous 옵션을 사용하라. 

```shell
$ kubectl exec <pod> cat <logfile>
```

#### 컨테이너의 로그 및 기타 파일 복사 

아직 살펴보지 않은 kubectl cp 명령어를 사용해 로그 파일을 로컬 시스템에 복사할 수도 있다.   
컨테이너의 또는 컨테이너로 파일을 복사할 수 있다. 예를 들어 foo-pod 라는 파드와 단일 컨테이너에   
/var/log/foo.log에 파일이 있으면 아래의 명령어를 사용해 로컬 컴퓨터로 전상할 수 있다. 

```shell
$ kubectl cp foo-pod:/var/log/foo.log foo.log 

$ kubectl cp localfile foo-pod:/etc/remotefile 
```

#### 중앙 집중식 로깅 

프로덕션 시스템에서는 중앙집중식, 클러스터 전체 로깅 솔루션을 사용해 모든 로그를 수집해 중앙 위치에 영구적으로  
저장하길 원할 것이다. 이것으로 히스토리 로그를 검토하고 경향을 분석할 수 있다. 이것으로 히스토리 로그를 검토하고  
경향을 분석할 수 있다. 이러한 시스템이 없으면 파드의 로그는 파드가 존재하는 동안에만 사용할 수 있다.  
파드가 삭제되는 즉시 로그도 삭제된다.

 쿠버네티스는 어떤 종류의 중앙로깅도 제공하지 않는다. 중앙 집중식 스토리지와 모든 컨테이너 로그를 분석하는 구성 요소는 
보통 클러스터 안에서 일반 파드로 실행되는 추가적인 구성 요소로 제공되어야 한다.

#### 여러 줄로 구성된 로그 문구 처리

사람이 읽을 수 있는 로그를 표준 출력으로 계속 출력하면서도 JSON 로그 파일에 떠서 플루언트디에서 처리하게 하는 것이다.  

# 쿠버네티스의 확장 
## 사용자 정의 API 오브젝트 정의 

쿠버네티스 생태계가 진화함에 따라 쿠버네티스가 현재 지원하는 리소스보다 훨씬더 높은 수준의 오브젝트가 점점 더 많이 나타나고 있다.  
디플로이먼트, 서비스, 컨피그 맵 등을 처리하는 대신 전체 애플리케이션이나 소프트웨어 서비스를 나타내는 오브젝트를 생성하고 관리한다.  
사용자 정의 컨트롤러는 이러한 높은 수준의 오브젝트를 관찰하과 이를 기반으로 낮은 수준의 오브젝트를 생성한다. 

따라서 각 사용자 정의 리소스를 추가하는 방법을 제공하고 있다.  

### CustomResourceDefinition 소개 

새로운 리소스 유형을 정의하려면 CustomResourceDefinition 오브젝트를 쿠버네티스 API 서버에 게시하면 된다.   
CustomResourceDefinition 오브젝트는 사용자 정의 리소스 유형에 관한 설명이다. CRD 가 게시되면 사용자는 다른 쿠버네티스  
리소스와 마찬가지로 JSON 또는 YAML 매니페스트를 API 서버에 게시해 사용자 정의 리소스 인스턴스를 만들 수 있다.

### 예제 소개 

쿠버네티스 클러스터에서 사용자가 파드, 서비스와 기타 쿠버네티스 리소스를 처리할 필요 없이 정적 웹사이트를 최대한 쉽게 실행할 수 있게 한다고  
가정해보자. 여기서 하고자 하는 것은 사용자가 웹 사이트 이름과 웹사이트 파일을 얻을 수 있는 소스를 초함하는 Website 유형의 오브젝트를 만드는 것이다.  
깃 리포지토리를 해당 파일의 소스로 사용한다. 사용자가 WebSite 리소스의 인스턴스를 만들면 쿠버네티스가 새 웹 서버 파드를 기동하고 서비스를 통해  
노출하도록 한다.  

```yaml
kind: Website 
metadata: 
  name: kubia 
spec: 
  gitRepo: https://github.com/luksa/kubia-website-example.git
```

### CustomResourceDefinition 오브젝트 생성 

쿠버네티스가 사용자 정의 Website 리소스 인스턴스를 허용하게 하려면 작성한 CustomResourceDefinition을 API 서버에 게시해야 한다.  

```yaml
apiVersion: apiextention.k8s.io/v1beta1 
kind: CustomResourceDefinition # CustomResourceDefinition 은 이 API 그룹과 버전에 속한다. 
metadata: 
  name: websites.extensions.example.com # 사용자 정의 오브젝트의 전체 이름 
spec: 
  scope: Namespaced  # Website 리소스가 네임스페이스 범위로 지정되길 원한다. 
  group: extension.example.com # Website 리소스의 API 그룹과 버전을 정의한다. 
  version: v1 
  names: 
    kind: Website # 사용자 정의 오브젝트 이름의 다양한 형태를 지정한다, 
    singular: websites 
    plural: websites
```

### 사용자 정의 리소스의 인스턴스 생성 

```yaml
apiVersion: extensions.example.com/v1 
kind: Website 
metadata: 
  name: kubia 
spec:
  ...
```

```shell
$ kubectl create -f kubia-website.yml 

$ kubectl get websites 
```

### 사용자 정의 리소스 인스턴스 검색 

```shell
$ kubectl get websites

$ kubectl get website kubia -o yaml 
```

### 사용자 정의 오브젝트의 인스턴스 삭제

```shell
$ kubectel delete website kubia 
```