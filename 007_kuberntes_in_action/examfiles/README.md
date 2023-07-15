# Exam

#### Exam 1
Create a new deployment test-deployment --image=busybox --replicas=2 

#### Exam 2
Create a new deployment called nginx-deploy, with image nginx:1.16 and 1 replica. Record the version. 
Next upgrade the deployment to version 1.17 using rolling update. 
Make sure there version update is recorded int the resource annotation 

#### Exam 3 

Create a new service "web-application"

#### Exam 4 

Create a Persistent Volume with the given specification. 

#### Exam 5 

Taint the worker node to be Unschedulable. Once done, create a pod called dev-redis, image redis:alpine to ensure  
workloads are not scheduled to this worker node. Finally, create a new pod called prod-redis and image redis:alpine  
with toleration to be scheduled on node01. 

#### Exam 6 

Set the node named worker node as unavailable and reschedule all the pods running on it. 

#### Exam 7 

Create a Pod called non-root-pod, image: redis:alpine

#### Exam 8 

Create a new service account with the name pvviewer. Grant this service account access to list all Persistent Volume in the  
cluster by creating an appropriated cluster role called pvviewer-role and ClusterRoleBinding called pvviewer-role-binding. 

#### Exam 9 

Create a NetworkPolicy which denies all ingress traffic 

#### Exam 10 

Create a pod myapp-pod and the use an initContainer that uses the busybox image and sleeps for 20 seconds 

#### Exam 11 

Create the ingress resource with name ingress-wear-watch to make the applications available at/wear on the  
Ingress service in app-space namespace. 

#### Exam 12

Schedule pod for node 

```shell
Name: nginx 
image: nginx 
Node Selector: disk=ssd 
```

#### Exam 13 

Create a new pod called super-user-pod with image busybox:1.28. Allow the pod to be able to set system_time. 
The container should sleep for 4800 seconds 

#### Exam 14 

Remove taint from the master node and verify node is untaint 

#### Exam 15 

Create a configmap called myconfigmap with literal value appname=myapp 

- ReplicaSet 을 만들어보기 
- Deployment 를 이용해 replicas 를 5로 변경한다. 
- kubectl 명령어를 이용해서 Pod의 replica를 1에서 5까지 증가시키는 명령어를 작성한다. 
- 디플로이먼트 롤아웃 일시 중지로 PodTemplateSpec에 여러 수정 사항을 적용하고, 재개하여 새로운 롤아웃을 시작한다.