## Sample CKAD Observability and Maintenance Q&A

### Application Observability and Maintenance – 15%

- Understand API deprecations
- Implement probes and health checks
- Use provided tools to monitor Kubernetes applications [\*\*](https://github.com/jamesbuckett/ckad-questions/blob/main/05-ckad-observability-maintenance.md#05-01-first-list-all-the-pods-in-the-cluster-by-cpu-consumption-then-list-all-the-pods-in-the-cluster-by-memory-consumption)
- Utilize container logs [\*\*](https://github.com/jamesbuckett/ckad-questions/blob/main/05-ckad-observability-maintenance.md#05-02-create-a-pod-called-log-pod-using-image-nginx-in-namespace-log-namespace-create-the-namespace-obtain-the-logs-for-the-nginx-pod-for-the-last-hour)
- Debugging in Kubernetes [\*\*](https://github.com/jamesbuckett/ckad-questions/blob/main/05-ckad-observability-maintenance.md#05-04-create-a-pod-called-json-pod-using-image-nginx-in-namespace-json-namespace-create-the-namespace-obtain-the-hostip-address-using-jsonpath)
<br />


<details class="faq box"><summary>Optional - Install a Sample Microservices Application</summary>
<p>

This application is useful to see CPU and Memory for a microservices application.

```bash
kubectl create ns ns-demo
kubectl apply -n ns-demo -f "https://raw.githubusercontent.com/jamesbuckett/terraform-digital-ocean/master/prep/complete-demo.yaml"
kubectl apply -n ns-demo -f "https://raw.githubusercontent.com/jamesbuckett/terraform-digital-ocean/master/prep/hpa-demo.yaml"
kubectl wait -n ns-demo deploy frontend --for condition=Available --timeout=90s
```
</p>
</details>
<br />

#### 05-01. First list all the pods in the cluster by CPU consumption. Then list all the pods in the cluster by Memory consumption.

<details class="faq box"><summary>Solution - kubectl top pods -A --sort-by=cpu</summary>
<p>

```bash
clear
# Requires metrics server to be installed and working
# Similar to Linux top command but for pods
kubectl top pods -A --sort-by=cpu | more
```

Output:

```console
NAMESPACE                 NAME                                                    CPU(cores)   MEMORY(bytes)
default                   falco-pxf8g                                             51m 👈👈👈   55Mi
ns-loki                   loki-release-prometheus-server-6d4f4df478-9z2f8         38m          356Mi
ns-demo                   adservice-68444cb46c-jvc86                              23m          202Mi
ns-loki                   loki-release-promtail-prvvn                             13m          34Mi
ns-demo                   recommendationservice-b4cf8f489-xwv49                   13m          69Mi
...
```

</p>
</details>

<details class="faq box"><summary>Solution - kubectl top pods -A --sort-by=memory</summary>
<p>

##### Solution

```bash
clear
# Requires metrics server to be installed and working
# Similar to Linux top command but for pods
kubectl top pods -A --sort-by=memory | more
```

Output:

```console
NAMESPACE                 NAME                                                    CPU(cores)   MEMORY(bytes)
ns-loki                   loki-release-prometheus-server-6d4f4df478-9z2f8         11m          356Mi 👈👈👈
ns-demo                   adservice-68444cb46c-jvc86                              20m          202Mi
kube-system               cilium-gcnbl                                            6m           165Mi
kube-system               cilium-htrth                                            18m          163Mi
kube-system               cilium-8h6vd                                            5m           162Mi
kube-system               cilium-ml27n                                            11m          161Mi
...
```

</p>
</details>
<br />

#### 05-02. Create a pod called `log-pod` using image `nginx` in namespace `log-namespace`. Create the namespace. Obtain the `logs` for the nginx pod for the `last hour`.

<details class="faq box"><summary>Overview</summary>
<p>

![05-02](https://user-images.githubusercontent.com/18049790/136656169-85488092-140a-44ff-98d2-233f16842154.png)

</p>
</details>

<details class="faq box"><summary>Prerequisites</summary>
<p>

##### Prerequisites

```bash
mkdir -p ~/ckad/
clear
kubectl create namespace log-namespace
kubectl run log-pod --image=nginx -n log-namespace
kubectl config set-context --current --namespace=log-namespace
kubectl get all
```

</p>
</details>

<details class="faq box"><summary>Help</summary>
<p>

```bash
clear
kubectl logs -h | more
```

Output:

```console
Examples:
  # Return snapshot logs from pod nginx with only one container
  kubectl logs nginx

  # Return snapshot logs from pod nginx with multi containers
  kubectl logs nginx --all-containers=true

  # Return snapshot logs from all containers in pods defined by label app=nginx
  kubectl logs -l app=nginx --all-containers=true

  # Return snapshot of previous terminated ruby container logs from pod web-1
  kubectl logs -p -c ruby web-1

  # Begin streaming the logs of the ruby container in pod web-1
  kubectl logs -f -c ruby web-1

  # Begin streaming the logs from all containers in pods defined by label app=nginx
  kubectl logs -f -l app=nginx --all-containers=true

  # Display only the most recent 20 lines of output in pod nginx
  kubectl logs --tail=20 nginx

  # Show all logs from pod nginx written in the last hour
  kubectl logs --since=1h nginx 👈👈👈 This example matches most closely to the question: for the `last hour`

  # Show logs from a kubelet with an expired serving certificate
  kubectl logs --insecure-skip-tls-verify-backend nginx

  # Return snapshot logs from first container of a job named hello
  kubectl logs job/hello

  # Return snapshot logs from container nginx-1 of a deployment named nginx
  kubectl logs deployment/nginx -c nginx-1
```

</p>
</details>

<details class="faq box"><summary>Solution</summary>
<p>

```bash
clear
# Straight forward match in the examples
kubectl logs --since=1h log-pod
```

</p>
</details>
<br />

#### 05-03. Output all the events for all namespaces by creation date.

<details class="faq box"><summary>Overview</summary>
<p>

![05-03](https://user-images.githubusercontent.com/18049790/136656243-8d251a08-6411-48f8-b630-84c295254d6e.png)

</p>
</details>

<details class="faq box"><summary>Solution</summary>
<p>

kubernetes.io bookmark: [Viewing, finding resources](https://kubernetes.io/docs/reference/kubectl/cheatsheet/#viewing-finding-resources)

```bash
clear
kubectl get events -A --sort-by=.metadata.creationTimestamp
```

</p>
</details>
<br />

#### 05-04. Create a pod called `json-pod` using image `nginx` in namespace `json-namespace`. Create the namespace. Obtain the `hostIP` address using `JSONPath`.

<details class="faq box"><summary>Overview</summary>
<p>

![08-json-path](https://user-images.githubusercontent.com/18049790/141224991-a6e64661-685d-4b4f-8868-97dab51ad356.jpg)

</p>
</details>

<details class="faq box"><summary>Prerequisites</summary>
<p>

```bash
clear
kubectl create namespace json-namespace
kubectl run json-pod --image=nginx -n json-namespace
kubectl config set-context --current --namespace=json-namespace
kubectl get all
```

</p>
</details>

<details class="faq box"><summary>Solution - kubectl explain</summary>
<p>

```bash
clear
# kubectl explain pod.spec --recursive
# kubectl explain pod.status --recursive
kubectl explain pod.status | more
```

Output:

```console
KIND:     Pod
VERSION:  v1

RESOURCE: status <Object> 👈👈👈 First element: =.status

DESCRIPTION:
Most recently observed status of the pod. This data may not be up to date.
Populated by the system. Read-only. More info:
https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

     PodStatus represents information about the status of a pod. Status may
     trail the actual state of a system, especially if the node that hosts the
     pod cannot contact the control plane.

FIELDS:
conditions <[]Object>
Current service state of pod. More info:
https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle#pod-conditions

containerStatuses <[]Object>
The list has one entry per container in the manifest. Each entry is
currently the output of `docker inspect`. More info:
https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle#pod-and-container-status

ephemeralContainerStatuses <[]Object>
Status for any ephemeral containers that have run in this pod. This field
is alpha-level and is only populated by servers that enable the
EphemeralContainers feature.

hostIP <string> 👈👈👈 Second element: =.status.hostIP
IP address of the host to which the pod is assigned. Empty if not yet
scheduled.

```

</p>
</details>

<details class="faq box"><summary>Solution - kubectl get pod json-pod -o jsonpath=</summary>
<p>

Construct the search query to `hostIP`.

kubernetes.io bookmark:[JSONPath Support](https://kubernetes.io/docs/reference/kubectl/jsonpath/)

```bash
kubectl get pod json-pod -o jsonpath={.status.hostIP}
```
OR
```bash
kubectl get pod json-pod -o jsonpath={..hostIP}
```

</p>
</details>
<br />

#### 05-05. Run the preparation steps. A deployment called `my-revision-deployment` will be created in the namespace `revision-namespace`. Check the status of this deployment. Check the revision history of this deployment. Roll back to the last good working deployment. Roll back to the earliest revision. Verify that it is now working.

<details class="faq box"><summary>Prerequisites</summary>
<p>

```bash
clear
kubectl create namespace revision-namespace
kubectl config set-context --current --namespace=revision-namespace
kubectl create deployment my-revision-deployment --image=nginx:1.18.0 --replicas=2
kubectl rollout status deployment my-revision-deployment
kubectl set image deployment.apps/my-revision-deployment nginx=nginx:1.19.0 --record
kubectl rollout status deployment my-revision-deployment
kubectl set image deployment.apps/my-revision-deployment nginx=nginx:1.20.0 --record
kubectl rollout status deployment my-revision-deployment
kubectl set image deployment.apps/my-revision-deployment nginx=ngin:1.21.0 --record
clear
```

</p>
</details>

<details class="faq box"><summary>Solution</summary>
<p>

```bash
clear
#Situational Awareness 
kubectl get all 
```

```bash
# Examine events from Deployment 
kubectl describe deployment.apps/my-revision-deployment
```

```bash
# Get Deployment Revisions
kubectl rollout history deployment.apps/my-revision-deployment
```

```bash
# Fix the immediate problem
kubectl rollout undo deployment.apps/my-revision-deployment
```

```bash
# Go back further to an earlier revision
kubectl rollout undo deployment.apps/my-revision-deployment --to-revision=2
```

</p>
</details>
<br />

#### 05-06. Patch the deployment from the previous question: `my-revision-deployment` to have a `revisionHistoryLimit` size of 20.

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

#### 05-07. Run the code in the preparation section. Once the deployment is running alter the environmental variable `TIER=web` to `TIER=app`


<details class="faq box"><summary>Prerequisites</summary>
<p>

```bash
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: null
  name: set-env-namespace
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: set-env-namespace
  labels:
    app: nginx
spec:
  replicas: 3
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
        env:
        - name: TIER
          value: web
EOF
```

```bash
kubectl config set-context --current --namespace=set-env-namespace
kubectl get all
```

</p>
</details>
<br />

<details class="faq box"><summary>Solution</summary>
<p>

```bash
# Describe the Deployment 
kubectl describe deployment.apps nginx-deployment | grep -i env -A 1
```

```bash
# Set the env using kubectl set env 
kubectl set env deployment.apps nginx-deployment TIER=app
```

```bash
# Describe the Deployment 
kubectl describe deployment.apps nginx-deployment | grep -i env -A 1
```

</p>
</details>
<br />

#### Clean Up

<details class="faq box"><summary>Clean Up</summary>
<p>

```bash
yes | rm -R ~/ckad/
kubectl delete ns json-namespace --force
kubectl delete ns log-namespace --force
kubectl delete ns revision-namespace --force
```

</p>
</details>

_End of Section_
