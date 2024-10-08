- [Cheat Sheet](#cheat-sheet)
  - [Kubectl Create](#kubectl-create)
  - [Kubectl Get](#kubectl-get)
  - [Kubectl Run](#kubectl-run)
  - [Kubectl Expose](#kubectl-expose)
  - [Kubectl Delete](#kubectl-delete)
  - [Kubectl Apply](#kubectl-apply)
  - [Kubectl Annotate](#kubectl-annotate)
  - [Kubectl Autoscale](#kubectl-autoscale)
  - [Kubectl Debug](#kubectl-debug)
  - [Kubectl Diff](#kubectl-diff)
  - [Kubectl Edit](#kubectl-edit)
  - [Kubectl kustomize](#kubectl-kustomize)
  - [Kubectl Patch](#kubectl-patch)
  - [Kubectl Label](#kubectl-label)
  - [Kubectl Replace](#kubectl-replace)
  - [Kubectl Rollout](#kubectl-rollout)
  - [Kubectl Scale](#kubectl-scale)
  - [Kubectl Set](#kubectl-set)
  - [Kubectl Wait](#kubectl-wait)
  - [Kubectl Attach](#kubectl-attach)
  - [Kubectl Auth](#kubectl-auth)
  - [Kubectl Copy](#kubectl-copy)
  - [Kubectl Exec](#kubectl-exec)
  - [Kubectl Logs](#kubectl-logs)
  - [Kubectl Port Forward](#kubectl-port-forward)
  - [Kubectl Proxy](#kubectl-proxy)
  - [Kubectl Top](#kubectl-top)
  - [Kubectl Cluster-Info](#kubectl-cluster-info)
  - [Kubectl Cordon](#kubectl-cordon)
  - [Kubectl Drain](#kubectl-drain)
  - [Kubectl Taint](#kubectl-taint)
  - [Kubectl Uncordon](#kubectl-uncordon)
  - [Kubectl Config](#kubectl-config)
  - [Kubectl Version](#kubectl-version)
  - [Kubectl Explain](#kubectl-explain)
  - [Kubectl Options](#kubectl-options)


# Cheat Sheet 

> [Kubernetes Cheat Sheet](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-strong-getting-started-strong-)


## Kubectl Create 

```shell
kubectl create namespace <insert-namespace-name-here>
```

## Kubectl Get 

```shell
kubectl get pods

kubectl get pods -o wide

kubectl get replicationcontroller web

kubectl get deployments.v1.apps -o json

kubectl get -o json pod web-pod-13je7

kubectl get -f pod.yaml -o json

kubectl get -k dir/

kubectl get -o template pod/web-pod-13je7 --template={{.status.phase}}

kubectl get pod test-pod -o custom-columns=CONTAINER:.spec.containers[0].name,IMAGE:.spec.containers[0].image

kubectl get rc,services

kubectl get rc/web service/frontend pods/web-pod-13je7

kubectl get clusterrolebindings 
 
kubectl get custerroles 

# Name으로 정렬된 서비스의 목록 조회
kubectl get services --sort-by=.metadata.name

# 재시작 횟수로 정렬된 파드의 목록 조회
kubectl get pods --sort-by='.status.containerStatuses[0].restartCount'

# PersistentVolumes을 용량별로 정렬해서 조회
kubectl get pv --sort-by=.spec.capacity.storage

# app=cassandra 레이블을 가진 모든 파드의 레이블 버전 조회
kubectl get pods --selector=app=cassandra -o \
  jsonpath='{.items[*].metadata.labels.version}'

# 예를 들어 'ca.crt'와 같이 점이 있는 키값을 검색한다
kubectl get configmap myconfig \
  -o jsonpath='{.data.ca\.crt}'

# 밑줄(`_`) 대신 대시(`-`)를 사용하여 base64 인코딩된 값을 조회
kubectl get secret my-secret --template='{{index .data "key-name-with-dashes"}}'

# 모든 워커 노드 조회 (셀렉터를 사용하여 'node-role.kubernetes.io/control-plane'
# 으로 명명된 라벨의 결과를 제외)
kubectl get node --selector='!node-role.kubernetes.io/control-plane'

# 네임스페이스의 모든 실행 중인 파드를 조회
kubectl get pods --field-selector=status.phase=Running

# 모든 노드의 외부IP를 조회
kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="ExternalIP")].address}'

# 특정 RC에 속해있는 파드 이름의 목록 조회
# "jq" 커맨드는 jsonpath를 사용하는 매우 복잡한 변환에 유용하다. https://stedolan.github.io/jq/ 에서 확인할 수 있다.
sel=${$(kubectl get rc my-rc --output=json | jq -j '.spec.selector | to_entries | .[] | "\(.key)=\(.value),"')%?}
echo $(kubectl get pods --selector=$sel --output=jsonpath={.items..metadata.name})

# 모든 파드(또는 레이블을 지원하는 다른 쿠버네티스 오브젝트)의 레이블 조회
kubectl get pods --show-labels

# 어떤 노드가 준비됐는지 확인
JSONPATH='{range .items[*]}{@.metadata.name}:{range @.status.conditions[*]}{@.type}={@.status};{end}{end}' \
 && kubectl get nodes -o jsonpath="$JSONPATH" | grep "Ready=True"

# 클러스터에서 실행 중인 모든 이미지
kubectl get pods -A -o=custom-columns='DATA:spec.containers[*].image'

# `default` 네임스페이스의 모든 이미지를 파드별로 그룹지어 출력
kubectl get pods --namespace default --output=custom-columns="NAME:.metadata.name,IMAGE:.spec.containers[*].image"

 # "registry.k8s.io/coredns:1.6.2" 를 제외한 모든 이미지
kubectl get pods -A -o=custom-columns='DATA:spec.containers[?(@.image!="registry.k8s.io/coredns:1.6.2")].image'

# 이름에 관계없이 메타데이터 아래의 모든 필드
kubectl get pods -A -o=custom-columns='DATA:metadata.*'
```

## Kubectl Run 

Create and run a particular image in a pod.

```shell
kubectl run nginx --image=nginx

kubectl run hazelcast --image=hazelcast/hazelcast --port=5701

kubectl run hazelcast --image=hazelcast/hazelcast --env="DNS_DOMAIN=cluster" --env="POD_NAMESPACE=default"

kubectl run nginx --image=nginx --overrides='{ "apiVersion": "v1", "spec": { ... } }'

kubectl run -i -t busybox --image=busybox --restart=Never

kubectl run nginx --image=nginx -- <arg1> <arg2> ... <argN>

kubectl scale deployment <deployment-name> --replicas=3 -n <namespace>
```

## Kubectl Expose 

```shell
kubectl expose rc nginx --port=80 --target-port=8000

kubectl expose -f nginx-controller.yaml --port=80 --target-port=8000

kubectl expose pod valid-pod --port=444 --name=frontend

kubectl expose service nginx --port=443 --target-port=8443 --name=nginx-https

kubectl expose rc streamer --port=4100 --protocol=UDP --name=video-stream

kubectl expose rs nginx --port=80 --target-port=8000

kubectl expose deployment nginx --port=80 --target-port=8000
```

## Kubectl Delete 

```shell 
kubectl delete -k dir

kubectl delete pods,services -l name=myLabel

kubectl delete pod foo --force
```

## Kubectl Apply 

```shell
# Apply the configuration in pod.json to a pod
kubectl apply -f ./pod.json

# Apply resources from a directory containing kustomization.yaml - e.g. dir/kustomization.yaml
kubectl apply -k dir/


cat pod.json | kubectl apply -f -


kubectl apply -f - <<EOF
  ~~중략~~
EOF
```

## Kubectl Annotate 

```shell
kubectl annotate pods foo description='my frontend'
```

## Kubectl Autoscale 

```shell
kubectl autoscale deployment foo --min=2 --max=10
```

## Kubectl Debug 

```shell
kubectl debug mypod -it --image=busybox
```

## Kubectl Diff 

```shell
kubectl diff -f pod.json
```

## Kubectl Edit 

```shell
kubectl edit deployment/mydeployment -o yaml --save-config
```

## Kubectl kustomize 

```shell 
kubectl kustomize /home/config/production
```

## Kubectl Patch 

```shell 
kubectl patch -f node.json -p '{"spec":{"unschedulable":true}}
```

## Kubectl Label 

```shell
kubectl label pods foo unhealthy=true

kubectl label --overwrite pods foo status=unhealthy

kubectl label pods --all status=unhealthy

kubectl label -f pod.json status=unhealthy

kubectl label pods foo status=unhealthy --resource-version=1

kubectl label pods foo bar-
```

## Kubectl Replace 

```shell
kubectl replace -f ./pod.json
```


## Kubectl Rollout 


- kubectl rollout

```shell 
 kubectl rollout SUBCOMMAND
```

> kubectl rollout pause RESOURCE   
> kubectl rollout undo RESOURCE   
> kubectl rollout restart RESOURCE   
> kubectl rollout resum RESOURCE 


- kubectl rollout history 

```shell
kubectl rollout history deployment/abc
```

## Kubectl Scale 

```shell
kubectl scale --replicas=3 rs/foo

kubectl scale --current-replicas=2 --replicas=3 deployment/mysql
```

## Kubectl Set

```shell
kubectl set env deployment/registry STORAGE_DIR=/local

kubectl set env deployment/sample-build --list

kubectl set env deployment/sample-build STORAGE_DIR=/data -o yaml
```

## Kubectl Wait

```shell
kubectl wait --for=condition=Ready pod/busybox1

kubectl wait --for=condition=Ready=false pod/busybox1
```

## Kubectl Attach 

```shell
kubectl attach mypod


kubectl attach pod/{pod_id}
```


## Kubectl Auth 

```shell
kubectl auth can-i create pods

kubectl auth can-i '*' '*'

kubectl auth can-i --list --namespace=foo
```


## Kubectl Copy

```shell
kubectl cp /tmp/foo_dir <some-pod>:/tmp/bar_dir
```

## Kubectl Exec 

```shell
kubectl exec mypod -- date

kubectl exec mypod -c ruby-container -- date

kubectl exec mypod -i -t -- ls -t /usr

kubectl exec mypod -it /bin/sh 

kubectl exec --stdin --tty shell-demo -- /bin/bash

kubectl exec shell-demo -- ps aux
kubectl exec shell-demo -- ls /
kubectl exec shell-demo -- cat /proc/1/mounts

```

## Kubectl Logs 

```shell
kubectl logs nginx

kubectl logs -l app=nginx --all-containers=true

kubectl logs --tail=20 nginx

kubectl logs --since=1h nginx

kubectl logs job/hello

kubectl logs deployment/nginx -c nginx-1
```

## Kubectl Port Forward 

```shell 
kubectl port-forward pod/mypod 5000 6000

kubectl port-forward service/myservice 8443:https

kubectl port-forward --address 0.0.0.0 pod/mypod 8888:5000

kubectl port-forward pod/mypod :5000
```

## Kubectl Proxy 

```shell 
kubectl proxy --api-prefix=/custom/
```

## Kubectl Top 

```shell 
kubectl top node 

kubectl top pods 
```

## Kubectl Cluster-Info 

```shell 
kubectl cluster-info dump --output-directory=/path/to/cluster-state
```

## Kubectl Cordon 

```shell 
# Mark node "foo" as unschedulable
kubectl cordon foo
```

## Kubectl Drain 

```shell 
kubectl drain foo --grace-period=900
```

## Kubectl Taint 

```shell 
kubectl taint nodes foo dedicated=special-user:NoSchedule
```

## Kubectl Uncordon 

```shell 
kubectl uncordon foo
```

## Kubectl Config 

```shell
kubectl config current-context

kubectl config delete-cluster minikube

kubectl config delete-context minikube

kubectl config delete-user minikube

kubectl config get-clusters

# Using Kubectl to Retrieve User Accounts
# list include user name also
kubectl config get-contexts

kubectl config get-contexts my-context

kubectl config get-users

kubectl config rename-context old-name new-name 

kubectl config set clusters.my-cluster.server https://1.2.3.4

kubectl config set-context gce --user=cluster-admin

kubectl config set-credentials cluster-admin --client-key=~/.kube/admin.key

kubectl config unset current-context

kubectl config use-context minikube

kubectl config view

kubectl config set-context --current --namespace=ggckad-s2
```

## Kubectl Version 

```shell 
kubectl version
```

## Kubectl Explain 

```shell 
kubectl explain pods
```

## Kubectl Options 

```shell 
kubectl options
```