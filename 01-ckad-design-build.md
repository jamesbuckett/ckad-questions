## Sample CKAD Application Design and Build Questions and Answers

### Application Design and Build – 20%
* Define, build and modify container images **
* Understand Jobs and CronJobs
* Understand multi-container Pod design patterns (e.g. sidecar, init and others)
* Utilize persistent and ephemeral volumes **

#### 01-01. Create a namespace called `storage-namespace`. Create a Persistent Volume called `my-pv` with `5Gi` storage using hostPath `/mnt/my-host`. Create a Persistent Volume Claim called `my-pvc` with `2Gi` storage. Create a pod called `storage-pod` using the nginx image. Mount the Persistent Volume Claim onto `/my-mount` in `storage-pod`.

kubernetes.io: [Create a PersistentVolume](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolume)

<details><summary>show</summary>
<p>

```bash
kubectl create namespace storage-namespace
kubectl config set-context --current --namespace=storage-namespace
```

kubernetes.io: [Create a PersistentVolume](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolume)

```bash
# Create a YAML file for the PV
vi 01-01-pv.yml
```

```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv              # Change
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 5Gi           # Change
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/my-host"   # Change
```

```bash
kubectl apply -f 01-01-pv.yml
kubectl get pv
```
Output:
```bash
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM
my-pv     5Gi        RWO            Retain           Available
```

</p>
</details>

<details><summary>show</summary>
<p>

kubernetes.io: [Create a PersistentVolumeClaim](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolumeclaim)

```bash
# Create a YAML file for the PVC
vi 01-01-pvc.yml
```

```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc          # Change
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi      # Change
```

```bash
kubectl apply -f 01-01-pvc.yml
kubectl get pv
kubectl get pvc
```

Output:
```bash
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM
my-pv     5Gi        RWO            Retain           Bound       storage-namespace/my-pvc  # STATUS=Bound means the PV and PVC are linked

NAME     STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
my-pvc   Bound    my-pv    5Gi        RWO            manual         6s                     # STATUS=Bound means the PV and PVC are linked
```

</p>
</details>

<details><summary>show</summary>
<p>

kubernetes.io: [Create a Pod](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-pod)

```bash
# Create a YAML file for the Pod
vi 01-01-pod.yml
```

```bash
apiVersion: v1
kind: Pod
metadata:
  name: storage-pod                    # Change
spec:
  volumes:
    - name: my-volume
      persistentVolumeClaim:
        claimName: my-pvc              # Change
  containers:
    - name: my-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/my-mount"       # Change
          name: my-volume

```

```bash
kubectl apply -f 01-01-pod.yml
# Verify that the volume is mounted
kubectl describe pod storage-pod | grep -i Mounts -A1
# Or just kubectl describe pod storage-pod 
```

Output
```bash
    Mounts:
      /my-mount from my-volume (rw)    # Success
```

</p>
</details>

#### 01-02. Create a namespace called `pod-namespace`. Create a pod called `pod-1` using `nginx` image. The container in the pod should be named `container-1`.

<details><summary>show</summary>
<p>

```bash
clear
# Create the namespace
kubectl create namespace pod-namespace
```

```bash
clear
# Switch context into the namespace so that all subsequent commands execute inside that namespace.
kubectl config set-context --current --namespace=pod-namespace
```

```bash
clear
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

kubectl run nginx --image=nginx --dry-run=client ### This example matches most closely to the question.

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
kubectl run pod-1 --image=nginx --dry-run=client -o yaml > q01-02.yml
```

```bash
# Edit the YAML file to make required changes
# Use the Question number in case you want to return to the question for reference or for review
vi q01-02.yml
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

# vi edits
# / - find
# d$ - delete to end of line
# :u - undo on any error
# :wq - write and quit
```

```bash
# Apply the YAML file to the Kubernetes API server
kubectl apply -f q01-02.yml
```

```bash
clear
# Quick verification that the pod was created and is working
kubectl get pod --watch
```

</p>
</details>

#### 01-03. Create a container from the attached Dockerfile and index.html. Name the image `my-image`. Name the container `my-container`. Run the container exposing port `8080` on the host and port `80` on the container. Stop the container. Delete the container.

Create a file called index.html
```bash
vi index.html
```

Edit index.html with the following text.
```bash
Hardships often prepare ordinary people for an extraordinary destiny.
```

Create a file called Dockerfile
```bash
vi Dockerfile
```

Edit the Docker with to include the text below
```bash
FROM nginx:latest
COPY ./index.html /usr/share/nginx/html/index.html

```bash
clear
# Build the docker image
docker build -t my-image:v0.1 .
```

```bash
clear
# Run the docker image
docker run -it --rm -d -p 8080:80 --name my-container my-image
```

```bash
clear
# Verify Opertaion
curl localhost:8080
```

```bash
clear
# List all images
docker ps -a
```

```bash
clear
# Stop the Container
docker container stop my-container
```

```bash
clear
# Delete the Image
docker image rm my-image

#### Clean Up 

<details><summary>show</summary>
<p>

```bash
kubectl delete ns pod-namespace --force
kubectl delete ns deployment-namespace --force
kubectl delete ns edit-namespace --force
```

</p>
</details>

*End of Section*