## Sample CKAD API Q&A

#### 06-01. What is the current active namespace?

<details class="faq box"><summary>Solution - kubectl config get-contexts</summary>
<p>

```bash
# List all namespaces, but which is currently active?
clear
kubectl get namespace
```

```console
NAME              STATUS   AGE
cert-manager      Active   15d
default           Active   15d
knative-serving   Active   15d
kourier-system    Active   15d
kube-node-lease   Active   15d
kube-public       Active   15d
kube-system       Active   15d
ns-chaos          Active   15d
ns-cookies        Active   13d
ns-demo           Active   15d
ns-fluentbit      Active   7d20h
ns-goldilocks     Active   15d
ns-loki           Active   15d
ns-vpa            Active   15d
projectcontour    Active   15d
```

kubernetes.io bookmark: [Kubectl context and configuration](https://kubernetes.io/docs/reference/kubectl/cheatsheet/#kubectl-context-and-configuration)

```bash
# Get the current active namespace
kubectl config get-contexts
```

```bash
CURRENT   NAME                            CLUSTER                         AUTHINFO                              NAMESPACE
*         do-sgp1-digital-ocean-cluster   do-sgp1-digital-ocean-cluster   do-sgp1-digital-ocean-cluster-admin   ns-cookies 👈👈👈 # ns-cookies is the active namespace
```

</p>
</details>
<br />

#### 06-02. List all the Kubernetes resources that can be found inside a namespace. By name only.

<details class="faq box"><summary>Solution - kubectl api-resources --namespaced=true</summary>
<p>

kubernetes.io bookmark: [Not All Objects are in a Namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/#not-all-objects-are-in-a-namespace)

```bash
clear
kubectl api-resources --namespaced=true | more
```

Output:

```console
NAME                               SHORTNAMES                           APIVERSION                                  NAMESPACED   KIND
bindings                                                                v1                                          true         Binding
configmaps                         cm                                   v1                                          true         ConfigMap
endpoints                          ep                                   v1                                          true         Endpoints
...

# Do not need the additional supplied columns.

```

</p>
</details>

<details class="faq box"><summary>kubectl api-resources --namespaced=true -o name</summary>
<p>

##### Solution

```bash
clear
kubectl api-resources --namespaced=true -o name | more
```

Output:

```console
bindings
configmaps
endpoints
events
...
```

</p>
</details>
<br />

#### 06-03. Give the command to list out all the available API groups on your cluster. Then list out the API's in the `named` group.

<details class="faq box"><summary>Prerequisites</summary>
<p>

```bash
clear
# Use the kubectl proxy to provide credentials to connect to the API server
# kubectl proxy starts a local proxy service on port 8001
# kubectl proxy uses credentials from kubeconfig file

kubectl proxy &
```

</p>
</details>

<details class="faq box"><summary>Solution - All API groups</summary>
<p>

```bash
clear
# List all available API groups from the API server

# /api is called the core API's #👈👈👈
# /apis is called the named API's - going forward new features will be made available under this API #👈👈👈

curl http://localhost:8001 | more
```

Output:

```console
{
  "paths": [
    "/.well-known/openid-configuration",
    "/api",
    "/api/v1",
    "/apis",
    "/apis/",
    "/apis/admissionregistration.k8s.io",
    "/apis/admissionregistration.k8s.io/v1",
    "/apis/admissionregistration.k8s.io/v1beta1",
    "/apis/apiextensions.k8s.io",
    "/apis/apiextensions.k8s.io/v1",
    "/apis/apiextensions.k8s.io/v1beta1",
    "/apis/apiregistration.k8s.io",
    "/apis/apiregistration.k8s.io/v1",
    "/apis/apiregistration.k8s.io/v1beta1",
    "/apis/apps",
    "/apis/apps/v1",
    "/apis/authentication.k8s.io",
    "/apis/authentication.k8s.io/v1",
    "/apis/authentication.k8s.io/v1beta1",
    "/apis/authorization.k8s.io",
    "/apis/authorization.k8s.io/v1",
    "/apis/authorization.k8s.io/v1beta1",
    "/apis/autoscaling",
    "/apis/autoscaling/v1",
    "/apis/autoscaling/v2beta1",
    "/apis/autoscaling/v2beta2",
    "/apis/batch",
    "/apis/batch/v1",
    "/apis/batch/v1beta1",
    "/apis/certificates.k8s.io",
    "/apis/certificates.k8s.io/v1",
    "/apis/certificates.k8s.io/v1beta1",
    "/apis/coordination.k8s.io",
    "/apis/coordination.k8s.io/v1",
    "/apis/coordination.k8s.io/v1beta1",
    "/apis/discovery.k8s.io",
    "/apis/discovery.k8s.io/v1",
    "/apis/discovery.k8s.io/v1beta1",
    "/apis/events.k8s.io",
    "/apis/events.k8s.io/v1",
    "/apis/events.k8s.io/v1beta1",
    "/apis/extensions",
...
```

</p>
</details>

<details class="faq box"><summary>Solution - named API's</summary>
<p>

```bash
clear
# List all supported resource groups under the `named` (apis) group

curl http://localhost:8001/apis | grep "name" | more
```

Output:

```console
...
      "name": "apiregistration.k8s.io",
      "name": "apps",
      "name": "events.k8s.io",
      "name": "authentication.k8s.io",
      "name": "authorization.k8s.io",
      "name": "autoscaling",
      "name": "batch",
      "name": "certificates.k8s.io",
      "name": "networking.k8s.io",
      "name": "extensions",
      "name": "policy",
      "name": "rbac.authorization.k8s.io",
      "name": "storage.k8s.io",
      "name": "admissionregistration.k8s.io",
      "name": "apiextensions.k8s.io",
      "name": "scheduling.k8s.io",
      "name": "coordination.k8s.io",
      "name": "node.k8s.io",
      "name": "discovery.k8s.io",
      "name": "flowcontrol.apiserver.k8s.io",
      "name": "crd.projectcalico.org",
      "name": "projectcontour.io",
      "name": "metrics.k8s.io",
...
```

</p>
</details>
<br />

#### 06-04. Strategic Merge Patch Question
* Patch the deployment from the previous question: `my-revision-deployment` to have a `revisionHistoryLimit` size of `20` using a **Strategic Merge Patch**

<details class="faq box"><summary>Overview</summary>
<p>

##### Overview

kubernetes.io bookmark: [Use a strategic merge patch to update a Deployment](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/update-api-object-kubectl-patch/#use-a-strategic-merge-patch-to-update-a-deployment)


```bash
kubectl config set-context --current --namespace=revision-namespace
clear
kubectl explain deployment.spec
```

Output:

```console
KIND:     Deployment
VERSION:  apps/v1

RESOURCE: spec <Object>  👈👈👈 First element: =.spec

DESCRIPTION:
     Specification of the desired behavior of the Deployment.

     DeploymentSpec is the specification of the desired behavior of the
     Deployment.

FIELDS:
   minReadySeconds      <integer>
     Minimum number of seconds for which a newly created pod should be ready
     without any of its container crashing, for it to be considered available.
     Defaults to 0 (pod will be considered available as soon as it is ready)

   paused       <boolean>
     Indicates that the deployment is paused.

   progressDeadlineSeconds      <integer>
     The maximum time in seconds for a deployment to make progress before it is
     considered to be failed. The deployment controller will continue to process
     failed deployments and a condition with a ProgressDeadlineExceeded reason
     will be surfaced in the deployment status. Note that progress will not be
     estimated during the time a deployment is paused. Defaults to 600s.

   replicas     <integer>
     Number of desired pods. This is a pointer to distinguish between explicit
     zero and not specified. Defaults to 1.

   revisionHistoryLimit <integer>  👈👈👈 Second element: =.spec.revisionHistoryLimit
     The number of old ReplicaSets to retain to allow rollback. This is a
     pointer to distinguish between explicit zero and not specified. Defaults to
     10.

   selector     <Object> -required-
     Label selector for pods. Existing ReplicaSets whose pods are selected by
     this will be the ones affected by this deployment. It must match the pod
     template's labels.

   strategy     <Object>
     The deployment strategy to use to replace existing pods with new ones.

   template     <Object> -required-
     Template describes the pods that will be created.
```

</p>
</details>

<details class="faq box"><summary>Solution</summary>
<p>

```bash
# What is the current setting
kubectl get deployment my-revision-deployment -o jsonpath={.spec.revisionHistoryLimit}
```

```bash
# Create a file to hold the patch
vi ~/ckad/patch-file.yaml
```

```bash
spec: # 👈👈👈 First element: =.spec
  revisionHistoryLimit: 20 # 👈👈👈 Second element: =.spec.revisionHistoryLimit
```

```bash
kubectl patch deployment my-revision-deployment --patch "$(cat ~/ckad/patch-file.yaml)"
```

```bash
# Verify your work
kubectl get deployment my-revision-deployment -o jsonpath={.spec.revisionHistoryLimit}
```


</p>
</details>
<br />

_End of Section_
