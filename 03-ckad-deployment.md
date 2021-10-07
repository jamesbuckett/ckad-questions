## Sample CKAD Application Deployment Q&A

### Application Deployment – 20%

- Use Kubernetes primitives to implement common deployment strategies (e.g. blue/green or canary)
- Understand Deployments and how to perform rolling updates [\*\*](https://github.com/jamesbuckett/ckad-questions/blob/main/03-ckad-deployment.md#03-01-create-a-namespace-called-deployment-namespace-create-a-deployment-called-my-deployment-with-three-replicas-using-the-nginx-image-inside-the-namespace-expose-port-80-for-the-nginx-container-the-containers-should-be-named-my-container-each-container-should-have-a-memory-request-of-25mi-and-a-memory-limit-of-100mi)
- Use the Helm package manager to deploy existing packages

#### 03-01. Create a namespace called `deployment-namespace`. Create a Deployment called `my-deployment`, with `three` replicas, using the `nginx` image inside the namespace. Expose `port 80` for the nginx container. The containers should be named `my-container`. Each container should have a `memory request` of 25Mi and a `memory limit` of 100Mi.

<details><summary>show</summary>
<p>

##### Prerequisites

```bash
mkdir ~/ckad/
clear
# Create the namespace
kubectl create namespace deployment-namespace
```

```bash
clear
# Switch context into the namespace so that all subsequent commands execute inside that namespace.
kubectl config set-context --current --namespace=deployment-namespace
```

##### Help Examples

```bash
clear
# Run the help flag to get examples
# kubectl create deployment -h
kubectl create deploy -h | more
```

Output:

```
Examples:
  # Create a deployment named my-dep that runs the busybox image
  kubectl create deployment my-dep --image=busybox

  # Create a deployment with a command
  kubectl create deployment my-dep --image=busybox -- date

  # Create a deployment named my-dep that runs the nginx image with 3 replicas
  kubectl create deployment my-dep --image=nginx --replicas=3 👈👈👈 This example matches most closely to the question: `three` replicas

  # Create a deployment named my-dep that runs the busybox image and expose port 5701
  kubectl create deployment my-dep --image=busybox --port=5701 👈👈👈 This example matches most closely to the question: `port 80`
```

</p>
</details>

<details><summary>show</summary>
<p>

##### Solution

```bash
clear
# Using the best example that matches the question
kubectl create deployment my-deployment --image=nginx --replicas=3 --port=80 --dry-run=client -o yaml > q03-01.yml
```

```bash
# Edit the YAML file to make required changes
vi ~/ckad/q03-01.yml
```

kubernetes.io: [Meaning of memory](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#meaning-of-memory)

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
        ports:
        - containerPort: 80
        name: my-container  #👈👈👈 Change from nginx to my container
        resources:          #👈👈👈 From Meaning of memory link above
          requests:         #👈👈👈 From Meaning of memory link above
            memory: "25Mi"  #👈👈👈 From Meaning of memory link above
          limits:           #👈👈👈 From Meaning of memory link above
            memory: "100Mi" #👈👈👈 From Meaning of memory link above
status: {}
```

```bash
clear
# Apply the YAML file to the Kubernetes API server
kubectl apply -f ~/ckad/q03-01.yml
```

```bash
clear
# Quick verification that the deployment was created and is working
kubectl get all
```

Output:

```
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

#### 03-02. In the previous question a Deployment called `my-deployment` was created. Allow network traffic to flow to this deployment from inside the cluster on `port 8080`.

<details><summary>show</summary>
<p>

##### Help Examples

```bash
clear
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
  kubectl expose deployment nginx --port=80 --target-port=8000 👈👈👈 This example matches most closely to the question.

```

</p>
</details>

<details><summary>show</summary>
<p>

##### Solution

```bash
clear
# Using the best example that matches the question
kubectl expose deployment my-deployment --port=8080 --target-port=80
```

Watch out for the statement from inside the Cluster so this is of type: ClusterIP

Types include:

- ClusterIP (default)
- NodePort
- LoadBalancer
- ExternalName

```bash
clear
# Check that the Service was created
  # Inside the namespace: my-deployment
  # Outside the namespace: my-deployment.deployment-namespace.svc.cluster.local
kubectl get service
```

Output:

```
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
my-deployment   ClusterIP   10.245.79.74   <none>        80/TCP    103s
```

```bash
clear
# A quicker check is to see if the Pod Endpoints are being load balanced
kubectl get endpoints
kubectl get pods -o wide
```

Output:

```
NAME            ENDPOINTS                                         AGE
my-deployment   10.244.0.250:80,10.244.1.132:80,10.244.1.246:80   5m20s
# The three replicas internal endpoints are registered
```

</p>
</details>

#### 03-03. Create a namespace called `edit-namespace`. Create a deployment called `edit-deployment` with `2` replicas using the `redis` image in namespace. After the deployment is running, alter the containers to use the `nginx` image. Then alter the containers to use the `nginx:1.14.2` image and record the change.

<details><summary>show</summary>
<p>

##### Prerequisites

```bash
clear
kubectl create namespace edit-namespace
kubectl create deployment edit-deployment --image=redis --replicas=2 -n edit-namespace
kubectl config set-context --current --namespace=edit-namespace
```

</p>
</details>

<details><summary>show</summary>
<p>

##### Solution

```bash
kubectl edit deployment.apps/edit-deployment
```

```bash
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2021-09-24T06:23:27Z"
  generation: 1
  labels:
    app: edit-deployment
  name: edit-deployment
  namespace: edit-namespace
  resourceVersion: "7856"
  uid: d482067c-da5f-43ce-aa31-25defd2d0de3
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
      - image: redis #👈👈👈 Change this to nginx
        imagePullPolicy: Always
        name: redis #👈👈👈 This is the catch, when you created the deployment it used the image=redis to also name the container redis
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
```

```bash
clear
# Check the image in the Deployment
kubectl describe deployment edit-deployment | grep Image
```

This works but does not record what the change was.

</p>
</details>

<details><summary>show</summary>
<p>

##### Solution

kubernetes.io:[Updating a Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment)

```bash
clear
# Use the kubectl set image command
kubectl set image deployment.apps/edit-deployment redis=nginx:1.14.2 --record
```

```bash
clear
# Check the image in the Deployment
kubectl describe deployment edit-deployment | grep Image
# Check that the change was recorded
kubectl rollout history deployment.apps/edit-deployment
```

</p>
</details>

#### Clean Up

<details><summary>show</summary>
<p>

```bash
yes | rm -R ~/ckad/
kubectl delete ns deployment-namespace --force
kubectl delete ns edit-namespace --force
```

</p>
</details>

_End of Section_
