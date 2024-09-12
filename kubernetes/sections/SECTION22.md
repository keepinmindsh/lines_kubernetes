- [Job](#job)

# Job 

```shell
kubectl create job my-job --image=busybox
```

```shell
kubectl create job my-job --image=busybox -- date
```

```shell
kubectl create job test-job --from=cronjob/a-cronjob
```