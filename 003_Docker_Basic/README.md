
# Docker 기본 명령어

### Docker 버전 체크

![](https://keepinmindsh.github.io/lines/assets/img/docker_001.png){: .align-center}

Docker의 명령은 docker {명령} 형식이며, Root 권한으로 이용해야 한다.

```shell

$ docker --version

```

Docker는 Docker Hub를 통해 이미지를 공유하는 생태계가 구축되어 았다. 리눅스 배포 판 부터 오픈 소스 기반의 환경이 구축되어 있는 Image를 제공하고 있다. docker search를 이용한 검색

### Pull 명령으로 이미지 받기

```shell

$ docker search nodejs
$ docker pull ubuntu:latest

```

![](https://keepinmindsh.github.io/lines/assets/img/docker_002.png){: .align-center}

pull 명령으로 이미지 받기   
docker pull {이미지 이름}:{태그}

### Images 명령으로 이미지 출력하기

```shell

 $ docker images / docker image ls

```

![](https://keepinmindsh.github.io/lines/assets/img/docker_003.png){: .align-center}

![](https://keepinmindsh.github.io/lines/assets/img/docker_004.png){: .align-center}

### run 명령어로 컨테이너 생성하기

```shell

$ docker run -i -t --name hello ubuntu bin/bash

```

![](https://keepinmindsh.github.io/lines/assets/img/docker_005.png){: .align-center}

이미지를 컨테이너로 생성한 후에 Bash 셀로 ubuntu에 접근한다.

docker run {옵션} {이미지 이름} {실행할 파일}  
-i(interactive) -t(Pseudo-try) : Bash 셀에 대한 입력 및 출력을 할 수 있다.  
--name : 컨테이너의 이름을 지정한다.  
여기에서 docker run을 통해 접속할 때 bin/bash를 바로 실행하여 들어갔기 때문에 exit 를 통해서 빠져나오면 컨테이너가 정지됩니다.

```shell

# docker 에서 외부로 연결할 포트를 지정할 경우 
# docker container의 6379포트외 외부 80포트를 연결해준다.
$ docker run -d -p 6379:80 --name lines-redis redis-6379:0.1

$ docker run -it -p 5005:5005 ubuntu

# run : 기동  , --name : 이름 지정 , -d : 백그라운드 기동 , -p : 외부/내부 포트 연결 , redis : 이미지 명 
$ docker run --name line-redis -d -p 6379:6379 redis

```

### ps 명령으로 컨테이너 목록 확인하기

ps 명령으로 컨테이너 목록 확인하기  
docker ps 형식으로 -a 옵션을 사용하면 정지된 컨테이너까지 모두 출력하고, 옵션을 사용하지 않으면 실행되고 있는 컨테이너만 출력합니다

```shell

$ docker ps -a / docker container ls 

```

![](https://keepinmindsh.github.io/lines/assets/img/docker_006.png){: .align-center}

### start 명령으로 컨테이너 시작하기

start 명령으로 컨테이너 시작하기  
docker start를 통해 container를 기동 시키고, docker ps 로 해당 실행된 컨테이너 상태를 확인한다.

```shell

$ docker start hello      

```

### restart 명령으로 컨테이너 시작하기

restart 명령으로 컨테이너 시작하기  
docker restart를 통해 container를 기동 시키고, docker ps 로 해당 실행된 컨테이너 상태를 확인한다.

```shell

$ docker restart hello

```

### attach 명령으로 컨테이너 시작하기

attach 명령으로 컨테이너 시작하기  
bin/bash를 통해 실행되기 때문에 명령을 자유롭게 입력할 수 있지만, DB나 서버 애플리케이션을 실행하면 입력은 할 수 없고, 출력만 보게된다.  
exit 또는 ctrl + D 를 통해 종료 가능

```shell

$ docker attach hello  

```

### exec 명령으로 외부에서 컨테이너 안의 명령 실행하기

Excel 명령으로 외부에서 컨테이너 안의 명령 실행하기  
docker exec {컨테이너 이름} {명령} {매개 변수}  
컨테이너가 실행되고 있는 상태에서만 사용할 수 있으며 정지된 상태에서는 사용할 수 없습니다.

```shell 

$ docker exec hello echo "Hello World"     

```

### stop 명령어로 컨테이너 정지하기

Stop 명령으로 컨테이너 정지하기  
docker stop {컨테이너 이름}

```shell

$ docker stop hello 

```

### rm 명령으로 컨테이너 삭제하기

rm 명령으로 컨테이너 삭제하기  
docker stop {컨테이너 이름}

```shell

$ docker rm hello    

```

### rmi 명령으로 이미지 삭제하기

rmi 명령으로 이미지 삭제하기  
docker stop {이미지 이름}:{태그}

```shell

$ docker rm hello  

```


### hello-python:0.1 이라는 이미지가 생성되었고 이를 실행 시켜 본다.

![](https://keepinmindsh.github.io/lines/assets/img/docker_images.png){: .align-center}

```shell

$ docker run --name hello-flask -d -p 4000:80 hello-python:0.1  

```

기동을 하였을 때, 아래와 같은 이미지가 표시된다.
- -d detached mode 흔히 말하는 백그라운드 모드
- -p 호스트와 컨테이너의 포트를 연결 (포워딩)
- -v 호스트와 컨테이너의 디렉토리를 연결 (마운트)
- -e 컨테이너 내에서 사용할 환경변수 설정
- –name 컨테이너 이름 설정
- –rm 프로세스 종료시 컨테이너 자동 제거
- -it -i와 -t를 동시에 사용한 것으로 터미널 입력을 위한 옵션
- –link 컨테이너 연결 [컨테이너명:별칭]

![](https://keepinmindsh.github.io/lines/assets/img/dockerfile_run.png){: .align-center}

### History 명령으로 이미지의 히스토리를 살펴보기

History 명령으로 이미지의 히스토리를 살펴보기  
docker hsitory <이미지 이름>:<태그>

```shell

$ docker history hello:0.1

```

### 내가 생성한 이미지의 컨테이너로 부터 파일을 꺼내고 싶을 때

내가 생성한 이미지의 컨테이너로 부터 파일을 꺼내고 싶을 때  
docker cp <컨테이너이름>:<경로> <호스트경로>

```shell

$ docker cp hello-nginx:/etc/nginx/nginx.conf ./

```

![](https://keepinmindsh.github.io/lines/assets/img/docker_007.png){: .align-center}

### Commit 명령으로 컨테이너의 변경사항을 이미지로 생성하기

Commit 명령으로 컨테이너의 변경사항을 이미지로 생성하기  
docker commit <옵션> <컨테이너 이름> <이미지 이름>:<태그>  
-a "Foo Bar <foo@bar.com>"  
-m "add hello.txt"

![](https://keepinmindsh.github.io/lines/assets/img/docker_008.png){: .align-center}

### diff명령으로 컨테이너에서 변경된 파일 확인하기

diff 명령으로 컨테이너에서 변경된 파일 확인하기  
docker diff <컨테이너 이름>  
A : 추가한 파일  
B : 변경된 파일  
C : 삭제된 파일

```shell

$ docker diff hello

```

![](https://keepinmindsh.github.io/lines/assets/img/docker_009.png){: .align-center}

### inspect 명령으로 세부 정보 확인하기

inspect 명령으로 세부 정보 확인하기  
docker inspect <이미지 또는 컨테이너 이름>

```shell

$ docker inspect hello

```

![](https://keepinmindsh.github.io/lines/assets/img/docker_010.png){: .align-center}

### Docker Logs - 컨네이너에서 발생하는 다양한 로그 확인

컨테이너의 로그를 확인하는 명령은 docker logs를 사용한다. docker logs 명령은 컨테이너의 표준 출력을 표시함으로써 애플리케이션 상태를 알 수 있다.

```shell

$ docker logs mariadb

# 컨테이너의 로그가 너무 많아 읽기 힘들면 --tail 옵션을 사용하여 마지막 로그부터 지정한 라인까지 출력할 수 있다. 
$ docker logs --tail 10 mariadb 


# --since 옵션은 입력받은 유닉스 시간 이후의 로그를 확인할 수 있으며 -t 옵션으로 타임 스탬프를 표시할 수도 있다. 
$ docker logs --since 15491503000 -t mariadb 

# -f 옵션을 이용하여 실시간 로그 확인 
$ docker logs --tail 10 -f mariadb                       

```

**로깅 드라이버**  
로깅 드라이버는 기본적으로 json-file 로 설정되지만 도커 데몬 시작 옵션에서 --log-drive

```shell 

$ docker run -it --log-driver none alpine ash

# 실행중인 컨테이너의 로깅 드라이버를 확인하려면 docker inspect 명령을 실행한다. 
$ docker inspect -f '{{.HostConfig.LogConfig.Type}}' <CONTAINER>

$ docker inspect -f '{{.HostConfig.LogConfig.Type}}' mariadb

# 컨테이너 로그는 json 형태로 docker 내부에 저장된다. 
$ /var/lib/docker/cotainers/${CONTAINER_ID}/${CONTAINER_ID}-json.log

# docker 명령을 통해 직접 이용할 경우 
$ docker run --log-opt max-size=100m --log-opt max-file=5 my-lines:lastest

```
