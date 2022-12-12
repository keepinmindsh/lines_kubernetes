
# Object - Service

![matchExpressions](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/service.png)

## ClusterIP

- 클러스터내 접근 가능!
    - 인가된 사용자 ( 운영자 )
        - 내부 관리자 접근 관리의 용도
    - 내부 대쉬보드
    - Pod의 서비스 상태 디버깅

```yaml
apiVersion: v1
kind: Service 
metadata: 
  name: svc-1
spec: 
  selector: 
    app: pod 
  ports: 
    - port: 9000
      target: 8080
type: ClusterIP
```

## NodePort

- 모든 Node 에 Port 할당
    - 내부 포트 서비스를 만들어서 서로 접근이 가능하게 할 수 있음
- 내가 다수 노드가 있는 서비스에 대해서 1번 노드로만 전달하더라도 연결되어 있는 각 파드로 자동으로 분산되어 호출됨.
    - 하지만 만약 내가 원하는 IP에 대해서만 설정하고 싶은 경우
        - externalTrafficPolicy: Local
- 내부망 연결용
    - 데모나 임시 연결용

```yaml
apiVersion: v1
kind: Service 
metadata: 
  name: svc-2 
spec: 
  selector: 
    app: pod 
  ports:
    - port:9000 
      tartPort: 8080 
      nodePort: 30000 #30000 ~ 32767 
type: NodePort 
```

## LoadBalancer

- 외부 접속을 할 경우 주로 많이 사용함.
- Load Balancer는 외부 IP를 연동하기 위한 설정이 필요
    - Plugin - GCP, AWS, Azure, OpenStack
- 외부 시스템 노출 용

```yaml
apiVersion: v1 
kind: Service 
metadata: 
  name: svc-3 
spec: 
  selector:
    app: pod 
  ports: 
    - port: 9000 
      targetPort: 8080
type: LoadBalancer 
```

## Headless

![matchExpressions](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/headless.png)

- FQDN

## EndPoint

![matchExpressions](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/endpoint.png)

Selector와 Label 없이 직접 Pod에 연결하는 방식으로 만들 수 있음

## External Name

![matchExpressions](https://github.com/keepinmindsh/lines_kubernetes/blob/main/assets/externalname.png)