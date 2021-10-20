## Sample CKAD Observability and Maintenance Q&A

### Application Observability and Maintenance – 15%

- Understand API deprecations
- Implement probes and health checks
- Use provided tools to monitor Kubernetes applications [\*\*](https://github.com/jamesbuckett/ckad-questions/blob/main/05-ckad-observability-maintenance.md#05-01-first-list-all-the-pods-in-the-cluster-by-cpu-consumption-then-list-all-the-pods-in-the-cluster-by-memory-consumption)
- Utilize container logs [\*\*](https://github.com/jamesbuckett/ckad-questions/blob/main/05-ckad-observability-maintenance.md#05-02-create-a-pod-called-log-pod-using-image-nginx-in-namespace-log-namespace-create-the-namespace-obtain-the-logs-for-the-nginx-pod-for-the-last-hour)
- Debugging in Kubernetes [\*\*](https://github.com/jamesbuckett/ckad-questions/blob/main/05-ckad-observability-maintenance.md#05-04-create-a-pod-called-json-pod-using-image-nginx-in-namespace-json-namespace-create-the-namespace-obtain-the-hostip-address-using-jsonpath)

#### 05-01. First list all the pods in the cluster by CPU consumption. Then list all the pods in the cluster by Memory consumption.

<details><summary>show</summary>
<p>

##### Solution

```bash
clear
# Requires metrics server to be installed and working
# Similar to Linux top command but for pods
kubectl top pods -A --sort-by=cpu | more
```

Output:

```
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

<details><summary>show</summary>
<p>

##### Solution

```bash
clear
# Requires metrics server to be installed and working
# Similar to Linux top command but for pods
kubectl top pods -A --sort-by=memory | more
```

Output:

```
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

#### 05-02. Create a pod called `log-pod` using image `nginx` in namespace `log-namespace`. Create the namespace. Obtain the `logs` for the nginx pod for the `last hour`.

<details><summary>show</summary>
<p>

##### Overview

![05-02](https://user-images.githubusercontent.com/18049790/136656169-85488092-140a-44ff-98d2-233f16842154.png)

</p>
</details>

<details><summary>show</summary>
<p>

##### Prerequisites

```bash
mkdir ~/ckad/
clear
kubectl create namespace log-namespace
kubectl run log-pod --image=nginx -n log-namespace
kubectl config set-context --current --namespace=log-namespace
kubectl get all
```

</p>
</details>

<details><summary>show</summary>
<p>

##### Help Examples

```bash
clear
kubectl logs -h | more
```

Output:

```
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

<details><summary>show</summary>
<p>

##### Solution

```bash
clear
# Straight forward match in the examples
kubectl logs --since=1h log-pod
```

</p>
</details>

#### 05-03. Output all the events for all namespaces by creation date.

<details><summary>show</summary>
<p>

##### Overview

![05-03](https://user-images.githubusercontent.com/18049790/136656243-8d251a08-6411-48f8-b630-84c295254d6e.png)

</p>
</details>

<details><summary>show</summary>
<p>

##### Solution

kubernetes.io bookmark: [Viewing, finding resources](https://kubernetes.io/docs/reference/kubectl/cheatsheet/#viewing-finding-resources)

```bash
clear
kubectl get events -A --sort-by=.metadata.creationTimestamp
```

</p>
</details>

#### 05-04. Create a pod called `json-pod` using image `nginx` in namespace `json-namespace`. Create the namespace. Obtain the `hostIP` address using `JSONPath`.

<details><summary>show</summary>
<p>

##### Overview

![05-04](https://user-images.githubusercontent.com/18049790/136656501-11f5b64c-a250-4e49-92cf-c729daf9cd34.png)

</p>
</details>

##### Prerequisites

<details><summary>show</summary>
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

<details><summary>show</summary>
<p>

##### Solution

```bash
clear
# kubectl explain pod.spec --recursive
# kubectl explain pod.status --recursive
kubectl explain pod.status | more
```

Output:

```
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

<details><summary>show</summary>
<p>

##### Solution

Construct the search query to `hostIP`.

kubernetes.io bookmark:[JSONPath Support](https://kubernetes.io/docs/reference/kubectl/jsonpath/)

```bash
kubectl get pod json-pod -o jsonpath={.status.hostIP}
```

</p>
</details>

</p>
</details>

#### Clean Up

<details><summary>show</summary>
<p>

```bash
yes | rm -R ~/ckad/
kubectl delete ns json-namespace --force
kubectl delete ns log-namespace --force
```

</p>
</details>

_End of Section_
