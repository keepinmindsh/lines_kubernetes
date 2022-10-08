# Object - Service 

## ClusterIP 

- 클러스터내 접근 가능! 

```yaml
apiVerison: v1
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

- 모든 Node에 Port 할당 
- 내가 다수 노드가 있는 서비스에 대해서 1번 노드로만 전달하더라도 연결되어 있는 각 파드로 자동으로 분산되어 호출됨. 
  - 하지만 만약 내가 원하는 IP에 대해서만 설정하고 싶은 경우 
    - externalTrafficPolicy: Local 


## LoadBalancer 