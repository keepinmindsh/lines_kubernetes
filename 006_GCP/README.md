
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