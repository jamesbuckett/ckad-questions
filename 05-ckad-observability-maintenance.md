## Sample CKAD Observability and Maintenance Q&A

### Application Observability and Maintenance – 15%

- Understand API deprecations
- Implement probes and health checks
- Use provided tools to monitor Kubernetes applications [\*\*](https://github.com/jamesbuckett/ckad-questions/blob/main/05-ckad-observability-maintenance.md#05-01-first-list-all-the-pods-in-the-cluster-by-cpu-consumption-then-list-all-the-pods-in-the-cluster-by-memory-consumption)
- Utilize container logs [\*\*](https://github.com/jamesbuckett/ckad-questions/blob/main/05-ckad-observability-maintenance.md#05-02-create-a-pod-called-log-pod-using-image-nginx-in-namespace-log-namespace-create-the-namespace-obtain-the-logs-for-the-nginx-pod-for-the-last-hour)
- Debugging in Kubernetes [\*\*](https://github.com/jamesbuckett/ckad-questions/blob/main/05-ckad-observability-maintenance.md#05-04-create-a-pod-called-json-pod-using-image-nginx-in-namespace-json-namespace-create-the-namespace-obtain-the-hostip-address-using-jsonpath)
<br />


#### 05-01. CPU and Memory of Pods Question
* First list **all** the pods in the cluster **sort by** CPU consumption. 
* Then list **all** the pods in the cluster **sort by** Memory consumption.

<details class="faq box"><summary>Prerequisite: Metrics Server - Kubernetes top command</summary>
<p>

Metrics Server installs into Kubernetes

By default the metrics server required for the `kubectl top` command is not present on Docker Desktop.

Please install the [metrics server](https://github.com/kubernetes-sigs/metrics-server) with the following command:

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

```bash
kubectl patch deployment metrics-server -n kube-system --type 'json' -p '[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--kubelet-insecure-tls"}]'
```

</p>
</details>

<details class="faq box"><summary>Optional - Install a Sample Microservices Application</summary>
<p>

This application is useful to see CPU and Memory for a [microservices application](https://github.com/GoogleCloudPlatform/microservices-demo).

```bash
kubectl create ns ns-demo
kubectl apply -n ns-demo -f "https://raw.githubusercontent.com/GoogleCloudPlatform/microservices-demo/master/release/kubernetes-manifests.yaml"
kubectl wait -n ns-demo deploy frontend --for condition=Available --timeout=90s
```
</p>
</details>

<details class="faq box"><summary>Solution</summary>
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

<details class="faq box"><summary>Solution</summary>
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

#### 05-02. Pod Logging Question
* Create a namespace called `log-namespace`. 
* Create a pod called `log-pod` using image `nginx` in namespace `log-namespace`. 
* Obtain the `logs` for the nginx pod for the `last hour`.

<details class="faq box"><summary>Prerequisites</summary>
<p>

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

#### 05-03. Kubernetes Events Question
* Output all the events for **all** namespaces by creation date.

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

#### 05-04. JSONPath Question

* Create a namespace called `json-namespace`. 
* Create a pod called `json-pod` using image `nginx` in namespace `json-namespace`. 
* Obtain the `hostIP` address using `JSONPath`.

<details class="faq box"><summary>Concepts</summary>
<p>

* JSONPath is a query language for JSON.
* It allows you to select and extract data from a JSON document. 

</p>
</details>

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
kubectl config set-context --current --namespace=json-namespace
kubectl run json-pod --image=nginx 
kubectl get all
```

</p>
</details>

<details class="faq box"><summary>Solution</summary>
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

<details class="faq box"><summary>Solution</summary>
<p>

Construct the search query to `hostIP`.

Search term: `JSONPath Support` on [kubernetes.io](https://kubernetes.io/)

kubernetes.io bookmark: [JSONPath Support](https://kubernetes.io/docs/reference/kubectl/jsonpath/)

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

#### 05-05. Deployment Revision History Question
* Run the preparation steps. 
* A deployment called `my-revision-deployment` will be created in the namespace `revision-namespace`. 
* Check the status of this deployment. 
* Check the revision **history** of this deployment. 
* **Undo** to the last good working deployment. 
* Roll back to the earliest revision. 
* Verify that it is now working.

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

#### 05-06. Convert manifests between different API versions.
* Updated the sample YAML file from `networking.k8s.io/v1beta1` to `networking.k8s.io/v1`

<details class="faq box"><summary>kubectl convert - Deal with deprecated API versions</summary>
<p>

Search for `kubectl convert install` and scroll until you find the `Install kubectl convert plugin` section for installation instructions.

The `kubectl convert` plugin installs into WSL Linux

Please install the [`kubectl convert` plugin](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-kubectl-convert-plugin) with the following commands:

```bash
curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl-convert
curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl-convert.sha256"
echo "$(<kubectl-convert.sha256) kubectl-convert" | sha256sum --check
sudo install -o root -g root -m 0755 kubectl-convert /usr/local/bin/kubectl-convert
kubectl convert --help
```

</p>
</details>

<details class="faq box"><summary>Prerequisites</summary>
<p>

Typical API deprecated warning message:
```console
Warning: policy/v1beta1 PodDisruptionBudget is deprecated in v1.21+, unavailable in v1.25+; use policy/v1 PodDisruptionBudget
poddisruptionbudget.policy/calico-kube-controllers created
```

```bash
vi ~/ckad/06-04-beta-ingress.yml
```

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: my-ingress #👈👈👈 Change: `my-ingress`
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: / #👈👈👈 Change
        pathType: Prefix
        backend:
          service:
            name: my-service #👈👈👈 Change: `my-service`
            port:
              number: 8080 #👈👈👈 Change: --port=8080
```

</p>
</details>

<details class="faq box"><summary>Solution</summary>
<p>

Search for `kubectl convert`

kubernetes.io bookmark: [Migrate to non-deprecated APIs](https://kubernetes.io/docs/reference/using-api/deprecation-guide/#migrate-to-non-deprecated-apis)

```bash
kubectl-convert -f ~/ckad/06-04-beta-ingress.yml --output-version networking.k8s.io/v1
```

Output:

```console
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
  creationTimestamp: null
  name: my-ingress
spec:
  rules:
  - http:
      paths:
      - backend: {}
        path: /
        pathType: Prefix
status:
  loadBalancer: {}
```

</p>
</details>
<br />


#### 05-07. Set Environment Variable Question
* Run the code in the preparation section. 
* Once the deployment is running alter the environmental variable `TIER=web` to `TIER=app`


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
