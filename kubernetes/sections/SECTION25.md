- [Kubectl Config](#kubectl-config)
  - [Cheat Sheet](#cheat-sheet)


# Kubectl Config 

## Cheat Sheet 

```shell
kubectl config current-context
```

```shell
kubectl config delete-cluster minikube
```

```shell
kubectl config delete-context minikube
```

```shell
kubectl config delete-user minikube
```

```shell
kubectl config get-clusters
```

```shell
kubectl config get-contexts
```

```shell
kubectl config get-contexts my-context
```

```shell
kubectl config get-users
```

```shell
kubectl config rename-context old-name new-name
```

```shell
kubectl config set clusters.my-cluster.server https://1.2.3.4
kubectl config set clusters.my-cluster.certificate-authority-data $(echo "cert_data_here" | base64 -i -)
kubectl config set contexts.my-context.cluster my-cluster
kubectl config set users.cluster-admin.client-key-data cert_data_here --set-raw-bytes=true
```

```shell
kubectl config set-cluster e2e --server=https://1.2.3.4
kubectl config set-cluster e2e --embed-certs --certificate-authority=~/.kube/e2e/kubernetes.ca.crt
kubectl config set-cluster e2e --insecure-skip-tls-verify=true
kubectl config set-cluster e2e --tls-server-name=my-cluster-name
```

```shell
kubectl config set-context gce --user=cluster-admin
```

```shell
kubectl config set-credentials cluster-admin --client-certificate=~/.kube/admin.crt --embed-certs=true
kubectl config set-credentials cluster-admin --auth-provider=gcp
kubectl config set-credentials cluster-admin --auth-provider=oidc --auth-provider-arg=client-id=foo --auth-provider-arg=client-secret=bar
kubectl config set-credentials cluster-admin --auth-provider=oidc --auth-provider-arg=client-secret-
kubectl config set-credentials cluster-admin --exec-command=/path/to/the/executable --exec-api-version=client.authentication.k8s.io/v1beta1
kubectl config set-credentials cluster-admin --exec-env=key1=val1 --exec-env=key2=val2
kubectl config set-credentials cluster-admin --exec-env=var-to-remove-
```

```shell
kubectl config use-context minikube
```

```shell
kubectl config view
kubectl config view --raw
kubectl config view -o jsonpath='{.users[?(@.name == "e2e")].user.password}'
```


> [https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#config](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#config)