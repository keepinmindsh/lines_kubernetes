# Kubectl 

##### Kubectl 명령어로 Pod에 접근하기 

```shell
$ kubectl exec request-pod -it /bin/bash 
``` 

##### nslookup으로 도메인 ip 조회하기 

```shell
$ nslookup {도메인명}
```

##### Kubectl 명령어를 활용한 상네 내용 확인하기 

```shell 
kubectl describe {컨트롤러명} {오브젝트명}
```