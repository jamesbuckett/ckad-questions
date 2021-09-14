## Sample CKAD Workload Questions and Answers

#### 01. List all the kubernetes resources that can be found in a namespace. By name only.

<details><summary>show</summary>
<p>

kubernetes.io: [Not All Objects are in a Namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/#not-all-objects-are-in-a-namespace)

```bash
kubectl api-resources --namespaced=true | more   
```

Output:
```bash
NAME                               SHORTNAMES                           APIVERSION                                  NAMESPACED   KIND
bindings                                                                v1                                          true         Binding
configmaps                         cm                                   v1                                          true         ConfigMap
endpoints                          ep                                   v1                                          true         Endpoints
...

# Do not need the additional supplied columns.

```

</p>
</details>

<details><summary>show</summary>
<p>

```bash
kubectl api-resources --namespaced=true -o name | more
```

Output:
```bash
bindings
configmaps
endpoints
events
...
```

</p>
</details>

#### 02. Create a pod called `pod-1` using image `nginx`, the container should be named `container-1` in the namespace `my-pod-namespace`. Create the namespace.

<details><summary>show</summary>
<p>

```bash
# Create the namespace
kubectl create namespace my-pod-namespace
```

```bash
# Switch context into the namespace so that all subsequent commands execute inside that namespace.
kubectl config set-context --current --namespace=my-pod-namespace
```

```bash
# Run the help flag to get examples
kubectl run -h | more
```

Output:
```bash
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

</p>
</details>

<details><summary>show</summary>
<p>

kubernetes.io: [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

```bash
# Using the best example that matches the question
kubectl run pod-1 --image=nginx --dry-run=client -o yaml > q2.yml
```

```bash
# Edit the YAML file to make required changes
# Use the Question number in case you want to return to the question for reference or for review
vi q2.yml
```

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

# Make edits
# d$ - delete to end of line
# :u - Undo on any error
# :wq - Write and Quit
```

```bash
# Apply the YAML file to the Kubernetes API server
kubectl apply -f q2.yml
```

```bash
# Quick verification that the pod was created and is working
kubectl get all
```

</p>
</details>

#### 03. Create a Deployment called `my-deployment`, with `three` replicas, using the `nginx` image. The containers should be named `my-container`. Each container should have a `memory request` of 25Mi and a `memory limit` of 100Mi. This deployment should run in the `my-deployment-namespace` namespace. Create the namespace.

<details><summary>show</summary>
<p>

```bash
# Create the namespace
kubectl create namespace my-deployment-namespace
```

```bash
# Switch context into the namespace so that all subsequent commands execute inside that namespace.
kubectl config set-context --current --namespace=my-deployment-namespace
```

```bash
# Run the help flag to get examples
# kubectl create deployment -h
kubectl create deploy -h | more
```

Output:
```bash
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

</p>
</details>

<details><summary>show</summary>
<p>

```bash
# Using the best example that matches the question
kubectl create deployment my-deployment --image=nginx --replicas=3 --dry-run=client -o yaml > q3.yml
```

```bash
# Edit the YAML file to make required changes
vi q3.yml
# Make edits
# d$ - delete to end of line
# :u - Undo on any error
# :wq - Write and Quit
```

[Meaning of memory](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#meaning-of-memory)

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: my-deployment
  name: my-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-deployment
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: my-deployment
    spec:
      containers:
      - image: nginx
        name: my-container  # Change from nginx to my container
        resources:          # From Meaning of memory link above          
          requests:         # From Meaning of memory link above
            memory: "25Mi"  # From Meaning of memory link above
          limits:           # From Meaning of memory link above
            memory: "100Mi" # From Meaning of memory link above
        resources: {}
status: {}
```

```bash
# Apply the YAML file to the Kubernetes API server
kubectl apply -f q3.yml
```

```bash
# Quick verification that the deployment was created and is working
kubectl get all
```

Output:
```bash
NAME                               READY   STATUS    RESTARTS   AGE
pod/my-deployment-67fc8546-9b4bm   1/1     Running   0          16m
pod/my-deployment-67fc8546-mjw24   1/1     Running   0          16m
pod/my-deployment-67fc8546-tp5bk   1/1     Running   0          16m

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-deployment   3/3     3            3           16m

NAME                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/my-deployment-67fc8546   3         3         3       16m
```

 </p>
</details>

#### 04. In the previous question a Deployment called `my-deployment` was created. Allow network traffic to flow to this deployment from inside the cluster.

<details><summary>show</summary>
<p>

```bash
# Run the help flag to get examples
kubectl expose -h | more
```

Output:
```
Examples:
  # Create a service for a replicated nginx, which serves on port 80 and connects to the containers on port 8000
  kubectl expose rc nginx --port=80 --target-port=8000

  # Create a service for a replication controller identified by type and name specified in "nginx-controller.yaml",
which serves on port 80 and connects to the containers on port 8000
  kubectl expose -f nginx-controller.yaml --port=80 --target-port=8000

  # Create a service for a pod valid-pod, which serves on port 444 with the name "frontend"
  kubectl expose pod valid-pod --port=444 --name=frontend

  # Create a second service based on the above service, exposing the container port 8443 as port 443 with the name
"nginx-https"
  kubectl expose service nginx --port=443 --target-port=8443 --name=nginx-https

  # Create a service for a replicated streaming application on port 4100 balancing UDP traffic and named 'video-stream'.
  kubectl expose rc streamer --port=4100 --protocol=UDP --name=video-stream

  # Create a service for a replicated nginx using replica set, which serves on port 80 and connects to the containers on
port 8000
  kubectl expose rs nginx --port=80 --target-port=8000

  # Create a service for an nginx deployment, which serves on port 80 and connects to the containers on port 8000
  kubectl expose deployment nginx --port=80 --target-port=8000

```

</p>
</details>

<details><summary>show</summary>
<p>

```bash
# Using the best example that matches the question
kubectl expose deployment my-deployment --port=80 --target-port=80
```

Watch out for the statement from inside the Cluster so this is of type: ClusterIP

- --type='': Type for this service: ClusterIP, NodePort, LoadBalancer, or ExternalName. Default is 'ClusterIP'.

```bash
# Check that the Service was created
kubectl get service
```

Output:
```bash
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
my-deployment   ClusterIP   10.245.79.74   <none>        80/TCP    103s
```

```bash
# A quicker check is to see if the Pod Endpoints are being load balanced
kubectl get endpoints
```

Output:
```bash
NAME            ENDPOINTS                                         AGE
my-deployment   10.244.0.250:80,10.244.1.132:80,10.244.1.246:80   5m20s
# The three replicas internal endpoints are registered
```

</p>
</details>

#### 05. Create a deployment called `edit-deployment` with `2` replicas using image `nginx` in namespace `edit-namespace`. Create the namespace. After deployment alter the containers to use the `redis` image and record the change.

<details><summary>show</summary>
<p>

```bash
kubectl create namespace edit-namespace
kubectl create deployment edit-deployment --image=nginx --replicas=2 -n edit-namespace
kubectl config set-context --current --namespace=edit-namespace
```

</p>
</details>

<details><summary>show</summary>
<p>

```bash
kubectl edit deployment.apps/edit-deployment
```

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2021-09-14T03:43:00Z"
  generation: 1
  labels:
    app: edit-deployment
  name: edit-deployment
  namespace: edit-namespace
  resourceVersion: "11650562"
  uid: fdf79d08-aaa7-40e7-92d4-71db9a8fb6b0
spec:
  progressDeadlineSeconds: 600
  replicas: 2
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: edit-deployment
  strategy:
      rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: edit-deployment
    spec:
      containers:
      - image: redis
        imagePullPolicy: Always
        name: nginx
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30

:wq # write and quit file
```
This does not work so switch to set the `set image` command

</p>
</details>

<details><summary>show</summary>
<p>

kubernetes.io:[Updating a Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment)

```bash
kubectl set image deployment.apps/edit-deployment nginx=redis --record
```

</p>
</details>

#### Clean Up 

```bash
kubectl delete ns my-pod-namespace 
kubectl delete ns my-deployment-namespace 
kubectl delete ns edit-namespace
```

*End of Section*