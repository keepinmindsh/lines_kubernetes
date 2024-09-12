- [Kubernetes 에서 디버깅할 수 있는 다양한 방법](#kubernetes-에서-디버깅할-수-있는-다양한-방법)
    - [노드의 쉘을 사용해서 디버깅하기](#노드의-쉘을-사용해서-디버깅하기)
    - [파드의 로그 확인하기](#파드의-로그-확인하기)
    - [exec를 통해 컨테이너 디버깅하기](#exec를-통해-컨테이너-디버깅하기)
    - [임시 디버그 컨테이너를 사용해서 디버깅하기](#임시-디버그-컨테이너를-사용해서-디버깅하기)
    - [애플리케이션 트러블 슈팅하기](#애플리케이션-트러블-슈팅하기)


# Kubernetes 에서 디버깅할 수 있는 다양한 방법 

### 노드의 쉘을 사용해서 디버깅하기

- [node shell debugging](https://kubernetes.io/ko/docs/tasks/debug/debug-application/debug-running-pod/#node-shell-session)

### 파드의 로그 확인하기 

```shell
$ kubectl logs ${POD_NAME} ${CONTAINER_NAME}

$ kubectl logs --previous ${POD_NAME} ${CONTAINER_NAME}
```

### exec를 통해 컨테이너 디버깅하기 

- [Container Debugging](https://kubernetes.io/ko/docs/tasks/debug/debug-application/debug-running-pod/#container-exec)

### 임시 디버그 컨테이너를 사용해서 디버깅하기 

- [ephemeral-container](https://kubernetes.io/ko/docs/tasks/debug/debug-application/debug-running-pod/#ephemeral-container)

### 애플리케이션 트러블 슈팅하기 

- [Application Trouble Shooting](https://kubernetes.io/ko/docs/tasks/debug/debug-application/)