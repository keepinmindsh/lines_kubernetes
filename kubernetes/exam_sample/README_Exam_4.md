# Exam 1 

```
kubectl config set-context --current --namespace=jsh-ns

vi depoyment-nginx.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80

kubectl create -f depoyment-nginx.yaml
deployment.apps/nginx-deployment created

vi service-nginx.yaml 
apiVersion: v1
kind: Service
metadata:
  name: service-nginx
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376

kubectl create -f service-nginx.yaml
service/service-nginx created

```

# Exam 2 

```
kubectl get pods nginx-deployment-7fb96c846b-lpnj9  -o custom-columns=NAME:.metadata.name,STATUS:.status.conditions[*].type
```

# Exam 3

```
apiVersion: app/v1
kind: PersistentVolumeClaim
metadata:
  name: pv-volume
  namespace: jsh-ns
spec:
  storageClassName: "" # Empty string must be explicitly set otherwise default StorageClass will be set
  volumeName: pv-volume
  accessModes: ReadWrite


kubectl create -f pvc-volumes.yaml
```

# Exam 4

```
kubectl create -f deployment-nginx-5.yaml
deployment.apps/nginx-deployment1 created
deployment.apps/nginx-deployment2 created
deployment.apps/nginx-deployment3 created
deployment.apps/nginx-deployment4 created
deployment.apps/nginx-deployment5 created


```