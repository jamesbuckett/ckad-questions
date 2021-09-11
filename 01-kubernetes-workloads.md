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

### 02. Create a pod called pod-1 using image nginx, the container should be named container-1

<details><summary>show</summary>
<p>

```bash
kubectl run pod-1 --image=nginx --dry-run=client -o yaml > pod-1.yaml
```

vi pod-1
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
  
</p>
</details>
