# Container 를 실전으로 해보기 

## 준비 사항 

- Docker Desktop 설치하기 
- Docker 명령어 동작 여부 확인하기 

## 실제 기동해보기 

```dockerfile
# syntax=docker/dockerfile:1
FROM golang:1.19-alpine

WORKDIR /app

COPY go.mod ./
COPY go.sum ./
RUN go mod download

COPY ./ ./
RUN go build -o /admin-backend

EXPOSE 8080

ENTRYPOINT ["/admin-backend"]
```

- dockerfiles 를 이용한 image 생성 처리 

```shell
$ docker build -t admin-backend-go .
```

- container 생성해보기 

```shell
$ docker run -d -p 8080:8080 admin-backend-go
```