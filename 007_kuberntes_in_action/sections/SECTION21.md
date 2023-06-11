# CronJob 

```shell
$ kubectl create cronjob my-job --image=busybox --schedule="*/1 * * * *"
```

```shell
$ kubectl create cronjob my-job --image=busybox --schedule="*/1 * * * *" -- date
```