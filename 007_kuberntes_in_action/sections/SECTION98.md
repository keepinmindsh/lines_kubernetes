# Cheat Sheet 

> [Kubernetes Cheat Sheet](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-strong-getting-started-strong-)


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

## Kubectl Edit 

```shell
kubectl edit deployment/mydeployment -o yaml --save-config
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

## Kubectl scale 

```shell
kubectl scale --replicas=3 rs/foo

kubectl scale --current-replicas=2 --replicas=3 deployment/mysql
```

## Kubectl set

```shell
kubectl set env deployment/registry STORAGE_DIR=/local

kubectl set env deployment/sample-build --list

kubectl set env deployment/sample-build STORAGE_DIR=/data -o yaml
```

## Kubectl wait

```shell
kubectl wait --for=condition=Ready pod/busybox1

kubectl wait --for=condition=Ready=false pod/busybox1
```

## Kubectl attach 

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