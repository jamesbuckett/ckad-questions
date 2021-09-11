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
kubectl run -h 

Examples:
  # Start a nginx pod
  kubectl run nginx --image=nginx

  # Start a hazelcast pod and let the container expose port 5701
  kubectl run hazelcast --image=hazelcast/hazelcast --port=5701

  # Start a hazelcast pod and set environment variables "DNS_DOMAIN=cluster" and "POD_NAMESPACE=default" in the
container
  kubectl run hazelcast --image=hazelcast/hazelcast --env="DNS_DOMAIN=cluster" --env="POD_NAMESPACE=default"

  # Start a hazelcast pod and set labels "app=hazelcast" and "env=prod" in the container
  kubectl run hazelcast --image=hazelcast/hazelcast --labels="app=hazelcast,env=prod"

  # Dry run; print the corresponding API objects without creating them
  kubectl run nginx --image=nginx --dry-run=client

  # Start a nginx pod, but overload the spec with a partial set of values parsed from JSON
  kubectl run nginx --image=nginx --overrides='{ "apiVersion": "v1", "spec": { ... } }'

  # Start a busybox pod and keep it in the foreground, don't restart it if it exits
  kubectl run -i -t busybox --image=busybox --restart=Never

  # Start the nginx pod using the default command, but use custom arguments (arg1 .. argN) for that command
  kubectl run nginx --image=nginx -- <arg1> <arg2> ... <argN>

  # Start the nginx pod using a different command and custom arguments
  kubectl run nginx --image=nginx --command -- <cmd> <arg1> ... <argN> 
```  
  
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
  
Examples:
  # Create a deployment named my-dep that runs the busybox image
  kubectl create deployment my-dep --image=busybox

  # Create a deployment with a command
  kubectl create deployment my-dep --image=busybox -- date

  # Create a deployment named my-dep that runs the nginx image with 3 replicas
  kubectl create deployment my-dep --image=nginx --replicas=3

  # Create a deployment named my-dep that runs the busybox image and expose port 5701
  kubectl create deployment my-dep --image=busybox --port=5701  
```

```bash
kubectl create deployment my-deployment --image=nginx --repliacs=3 -n my-deployment-namespace --dry-run=client -o yaml > q3.yml
```

`vi q3.yml`
  
```bash
kubectl create deployment my-deployment --image=nginx --repliacs=3 -n my-deployment-namespace --dry-run=client -o yaml > q3.yml
```
  
