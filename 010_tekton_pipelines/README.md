# Tekton Pipeline 

### Get Started 

> [Tekton Getstarted](https://tekton.dev/docs/getting-started/)   
> [Tekton CI/CD](https://hevodata.com/learn/tekton-ci-cd/)     
> [Building k8s tekton](https://earthly.dev/blog/building-k8s-tekton/)

### Tekton Hub 

> [Tekton Hub](https://hub.tekton.dev/)

### Begin 

```shell
$ kubectl apply --filename https://storage.googleapis.com/tekton-releases-nightly/pipeline/latest/release.yaml 
namespace/tekton-pipelines unchanged
clusterrole.rbac.authorization.k8s.io/tekton-pipelines-controller-cluster-access configured
clusterrole.rbac.authorization.k8s.io/tekton-pipelines-controller-tenant-access unchanged
clusterrole.rbac.authorization.k8s.io/tekton-pipelines-webhook-cluster-access configured
role.rbac.authorization.k8s.io/tekton-pipelines-controller unchanged
role.rbac.authorization.k8s.io/tekton-pipelines-webhook unchanged
role.rbac.authorization.k8s.io/tekton-pipelines-leader-election unchanged
role.rbac.authorization.k8s.io/tekton-pipelines-info unchanged
serviceaccount/tekton-pipelines-controller unchanged
serviceaccount/tekton-pipelines-webhook unchanged
clusterrolebinding.rbac.authorization.k8s.io/tekton-pipelines-controller-cluster-access unchanged
clusterrolebinding.rbac.authorization.k8s.io/tekton-pipelines-controller-tenant-access unchanged
clusterrolebinding.rbac.authorization.k8s.io/tekton-pipelines-webhook-cluster-access unchanged
rolebinding.rbac.authorization.k8s.io/tekton-pipelines-controller unchanged
rolebinding.rbac.authorization.k8s.io/tekton-pipelines-webhook unchanged
rolebinding.rbac.authorization.k8s.io/tekton-pipelines-controller-leaderelection unchanged
rolebinding.rbac.authorization.k8s.io/tekton-pipelines-webhook-leaderelection unchanged
rolebinding.rbac.authorization.k8s.io/tekton-pipelines-info unchanged
customresourcedefinition.apiextensions.k8s.io/clustertasks.tekton.dev configured
customresourcedefinition.apiextensions.k8s.io/customruns.tekton.dev configured
customresourcedefinition.apiextensions.k8s.io/pipelines.tekton.dev configured
customresourcedefinition.apiextensions.k8s.io/pipelineruns.tekton.dev configured
customresourcedefinition.apiextensions.k8s.io/resolutionrequests.resolution.tekton.dev unchanged
customresourcedefinition.apiextensions.k8s.io/pipelineresources.tekton.dev configured
customresourcedefinition.apiextensions.k8s.io/runs.tekton.dev configured
customresourcedefinition.apiextensions.k8s.io/tasks.tekton.dev configured
customresourcedefinition.apiextensions.k8s.io/taskruns.tekton.dev configured
customresourcedefinition.apiextensions.k8s.io/verificationpolicies.tekton.dev created
secret/webhook-certs configured
validatingwebhookconfiguration.admissionregistration.k8s.io/validation.webhook.pipeline.tekton.dev configured
mutatingwebhookconfiguration.admissionregistration.k8s.io/webhook.pipeline.tekton.dev configured
validatingwebhookconfiguration.admissionregistration.k8s.io/config.webhook.pipeline.tekton.dev configured
clusterrole.rbac.authorization.k8s.io/tekton-aggregate-edit unchanged
clusterrole.rbac.authorization.k8s.io/tekton-aggregate-view unchanged
configmap/config-artifact-bucket unchanged
configmap/config-artifact-pvc unchanged
configmap/config-defaults configured
configmap/feature-flags configured
configmap/pipelines-info configured
configmap/config-leader-election unchanged
configmap/config-logging unchanged
configmap/config-observability unchanged
configmap/config-registry-cert unchanged
configmap/config-trusted-resources unchanged
deployment.apps/tekton-pipelines-controller configured
service/tekton-pipelines-controller configured
namespace/tekton-pipelines-resolvers unchanged
clusterrole.rbac.authorization.k8s.io/tekton-pipelines-resolvers-resolution-request-updates unchanged
role.rbac.authorization.k8s.io/tekton-pipelines-resolvers-namespace-rbac unchanged
serviceaccount/tekton-pipelines-resolvers unchanged
clusterrolebinding.rbac.authorization.k8s.io/tekton-pipelines-resolvers unchanged
rolebinding.rbac.authorization.k8s.io/tekton-pipelines-resolvers-namespace-rbac unchanged
configmap/bundleresolver-config unchanged
configmap/cluster-resolver-config unchanged
configmap/resolvers-feature-flags unchanged
configmap/config-leader-election unchanged
configmap/config-logging unchanged
configmap/config-observability unchanged
configmap/git-resolver-config unchanged
configmap/hubresolver-config unchanged
deployment.apps/tekton-pipelines-remote-resolvers configured
horizontalpodautoscaler.autoscaling/tekton-pipelines-webhook configured
deployment.apps/tekton-pipelines-webhook configured
service/tekton-pipelines-webhook configured
``

```shell 
$ kubectl get pods --namespace tekton-pipelines --watch
tekton-pipelines-controller-77d76b7699-vg6w7   0/1     Running             0               16s
tekton-pipelines-controller-84f9885768-h9zqw   1/1     Running             8 (4m59s ago)   28d
tekton-pipelines-webhook-6dbd4f6f84-9scrv      0/1     ContainerCreating   0               16s
tekton-pipelines-webhook-b877d487c-rpbbk       1/1     Running             8 (4m59s ago)   28d
tekton-pipelines-controller-77d76b7699-vg6w7   1/1     Running             0               20s
tekton-pipelines-controller-84f9885768-h9zqw   1/1     Terminating         8 (5m3s ago)    28d
tekton-pipelines-controller-84f9885768-h9zqw   0/1     Terminating         8 (5m4s ago)    28d
tekton-pipelines-controller-84f9885768-h9zqw   0/1     Terminating         8 (5m4s ago)    28d
tekton-pipelines-controller-84f9885768-h9zqw   0/1     Terminating         8 (5m4s ago)    28d
tekton-pipelines-webhook-6dbd4f6f84-9scrv      0/1     Running             0               23s
...
```