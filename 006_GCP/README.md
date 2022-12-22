
# GCloud SDK 사용하기 

모든 가이드는 GCloud SDK를 설치했고, MacOS를 사용하는 기준으로 설명한다. 

#### gcloud 로그인 하기 

```shell 

$ gcloud auth login # 로그인하기 - 브라우저를 통해 로그인함. 

$ gcloud auth list # 로그인한 계정 목록 조회하기 

```

#### gcloud 프로젝트 세팅하기 

```shell 

$ gcloud config set project {Project ID of GCP} # Project 지정하기 

$ gcloud projects list 

```

### gcloud cli를 통한 kubernetes cluster 활성화 하기 

```shell 

$ gcloud services enable container.googleapis.com

Operation "operations/acf.p2-948099043330-c0f27b50-5523-4623-b28d-1cbe9e2b08ee" finished successfully.

```

- f1-micro 는 메모리가 불충분해서 Node 생성이 불가하다고 하여 e2-micro로 생성 처리 

```shell

$ gcloud container clusters create lines-admin --num-nodes 3 --machine-type e2-micro --region us-central1

```

# GCloud 사용 Tip 

#### GKE 사용을 위해서 gcloud 명령어를 통한 생성시 

```shell 

ERROR: (gcloud.container.clusters.create) ResponseError: code=400, message=Failed precondition when calling the ServiceConsumerManager: tenantmanager::185014: Consumer 948099043330 should enable service:container.googleapis.com before generating a service account.

```

- [해결 방법 가이드](https://stackoverflow.com/questions/64537546/error-gcloud-container-clusters-create-responseerror-code-400-message-faile)

container.googleapis.com 이라는 서비스를 활성화 시켜야 한다고 한다. 