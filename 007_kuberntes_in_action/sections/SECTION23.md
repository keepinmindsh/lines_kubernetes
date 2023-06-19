# PriorityClass 

```shell
kubectl create priorityclass high-priority --value=1000 --description="high priority"
```

```shell
kubectl create priorityclass default-priority --value=1000 --global-default=true --description="default priority"
```

```shell
kubectl create priorityclass high-priority --value=1000 --description="high priority" --preemption-policy="Never"
```