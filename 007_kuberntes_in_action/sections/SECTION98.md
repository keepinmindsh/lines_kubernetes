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