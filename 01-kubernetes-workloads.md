## Kubernetes Workloads

### 01. List all the namespaces in the cluster

<details><summary>show</summary>
<p>

```bash
kubectl get namespaces
k get ns  
```

</p>
</details>

### 02. Create a pod called `pod-1` using image `nginx`, the container should be named `container-1` in the namespace `my-namespace`

<details><summary>show</summary>
<p>

```bash
kubectl run pod-1 --image=nginx --dry-run=client -o yaml > q2.yml
```

`vi q2.yml`
  
```bash
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod-1
  name: pod-1
spec:
  containers:
  - image: nginx
    name: container-1 # Change from pod-1 to container-1
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

```

```bash
kubectl apply -f q2.yml -n my-namespace
```
  
</p>
</details>

### 03. Create a Deployment called `my-deployment`, with three replicas, using the `nginx` image. The containers should be named `my-container`. Each container should have a memory request of 20Mi and a memory limit of 50Mi. This deployment should run in the `my-deployment-namespace` namespace.

<details><summary>show</summary>
<p>

```bash
kubectl create deployment -h
kubectl create deploy -h
```

```bash
kubectl create deployment my-deployment --image=nginx --repliacs=3 -n my-deployment-namespace --dry-run=client -o yaml > q3.yml
```

`vi q3.yml`
  
```bash
kubectl create deployment my-deployment --image=nginx --repliacs=3 -n my-deployment-namespace --dry-run=client -o yaml > q3.yml
```
  
