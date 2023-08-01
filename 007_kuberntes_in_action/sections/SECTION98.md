# Cheat Sheet 

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