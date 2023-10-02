# Section 2 - Container 의 이해 ( feat. Docker )

```shell
$ docker run busybox echo "Hello world"
```

## 기본적인 dockerfile

```dockerfile
FROM node:7 
ADD app.js /app.js 
ENTRYPOINT ["node", "app.js"]
```

### 컨테이너 이미지 생성

```shell
$ docker build -t kubia . 
```

> 빌드 디렉터리에 불필요한 파일을 포함시키지 마라. 특히 원격 머신에 도커 데몬이 위치한 경우 이러한 파일이 빌드 프로세스의 속도 저하를 가져온다.

### 이미지 레이어

서로 다른 이미지가 여러 개의 레이어를 공유할 수 있기 때문에 이미지의 저장과 전송에 효과적이다. 예를 들어

- 동일한 기본이미지를 바탕으로 다수의 이미지를 생성하더라도 기본 이미지를 구성하는 모든 레이어는 단 한번만 저장될 것이다.
- 컴퓨터에 여러 개의 레이어가 이미 저장돼 있다면 도커는 저장되지 않은 레이어만 다운로드 한다.

각 Dockerfile이 새로운 레이어를 하나만 생성하다고 생각할 수 있지만 그렇지 않다.  
이미지를 빌드하는 동안 기본 이미지의 모든 레이어를 가져온다음, 도커는 그 위에 새로운 레이어를 생성하고 app.js 파일을 그위에 추가한다.  
그런 다음 이미지가 실행할 때 수행돼야 할 명령을 지정하는 또 하나의 레이어를 추가한다.  
이 마지막 레이어는 kubia:latest 라고 태그를 지정한다. 

### Docker Basic 

- [Docker Basic](https://github.com/keepinmindsh/lines_kubernetes/tree/main/003_Docker_Basic) 

### Docker Files 

- [Docker files](https://github.com/keepinmindsh/lines_kubernetes/tree/main/004_DockerFiles)

## CRI ( 쿠버네티스 컨테이너 런타임 ) / CTR 

2013년 Docker가 처음으로 세상에 나타난 이후 IT업계는 정말 크게 변했습니다. 어플리케이션의 배포단위는 이제 war, jar, zip 등이 아니라 Docker 이미지가 되었고 Docker를 사용할 수 있는 환경이기만 하면 어플리케이션은 Windows에서든, Ubuntu에서든 동일하게 동작하였습니다. Docker의 이러한 특성을 이용해 수백 수천대의 서버를 운영하는 환경에서 Docker를 도입하는 사례들이 늘어나기 시작했습니다.

하지만 포맷과 런타임에 대한 특정한 규격이 없다 보니 컨테이너의 미래는 불안했습니다. 도커가 사실상의 컨테이너 표준 역할을 했지만 코어OS(CoreOS)는 도커와는 다른 규격으로 표준화를 추진하려 했습니다.

이때 Docker Engine에는 API, CLI, 네트워크, 스토리지 등 많은 기능들이 하나의 패키지에 담겨있었고, Docker 측에서는 Monolithic한 Docker Engine의 구조를 나누는 작업을 시작했습니다.

- Docker 
- Container Runtime Interface > docker shim > Docker ( containerd : CLI, API, BUILD, VOLUMES, AUTH, SECURITY )

### CLI - ctr 
### CLI - nerdctl 
### CLI - crictl

> [https://kr.linkedin.com/pulse/containerd%EB%8A%94-%EB%AC%B4%EC%97%87%EC%9D%B4%EA%B3%A0-%EC%99%9C-%EC%A4%91%EC%9A%94%ED%95%A0%EA%B9%8C-sean-lee](https://kr.linkedin.com/pulse/containerd%EB%8A%94-%EB%AC%B4%EC%97%87%EC%9D%B4%EA%B3%A0-%EC%99%9C-%EC%A4%91%EC%9A%94%ED%95%A0%EA%B9%8C-sean-lee)
> [컨테이너 생태계에 대해서 잘 설명된 블로그 (참조)](https://blog.siner.io/2021/10/23/container-ecosystem/)
> [https://wookiist.dev/142](https://wookiist.dev/142)
> [https://ybchoi.com/22](https://ybchoi.com/22)

### socket 

- unix://var/run/dockershim.sock
- unix://run/containerd/containerd.sock
- unix://run/crio.sock 
- unix://var/run/cri-dockerd.sock 