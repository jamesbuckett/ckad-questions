## Sample CKAD Application Design and Build Q&A

### Application Design and Build – 20%

- Define, build and modify container images [\*\*](https://github.com/jamesbuckett/ckad-questions/blob/main/01-ckad-design-build.md#01-01-create-a-container-from-the-attached-dockerfile-and-indexhtml-name-the-image-my-image-name-the-container-my-container-run-the-container-exposing-port-8080-on-the-host-and-port-80-on-the-container-stop-the-container-delete-the-container)
- Understand Jobs and CronJobs
- Understand multi-container Pod design patterns (e.g. sidecar, init and others)
- Utilize persistent and ephemeral volumes [\*\*](https://github.com/jamesbuckett/ckad-questions/blob/main/01-ckad-design-build.md#01-03-create-a-namespace-called-storage-namespace-create-a-persistent-volume-called-my-pv-with-5gi-storage-using-hostpath-mntmy-host-create-a-persistent-volume-claim-called-my-pvc-with-2gi-storage-create-a-pod-called-storage-pod-using-the-nginx-image-mount-the-persistent-volume-claim-onto-my-mount-in-storage-pod)
<br />

#### 01-01. Docker Question
* Create a container from the attached `Dockerfile` and `index.html`. 
* Name the image `my-image`. 
* Run the container exposing port `8080` on the host and port `80` on the container. 
* Name the container `my-container`. Stop the container. Delete the container.

<details class="faq box"><summary>Docker Image Creation</summary>
<p>

Create a file called index.html

```bash
mkdir -p ~/ckad/
vi ~/ckad/index.html
```

Edit index.html with the following text.

```bash
Hardships often prepare ordinary people for an extraordinary destiny.
```

Create a file called Dockerfile

```bash
vi ~/ckad/Dockerfile
```

Edit the Docker with to include the text below
:
```bash
FROM nginx:latest
COPY ./index.html /usr/share/nginx/html/index.html
```

```bash
cd ~/ckad/
clear
# Build the docker image
docker build -t my-image:v0.1 .
docker images
```

```bash
# Create a TAR file from the image file
docker save --output my-image.tar my-image:v0.1
ls my-image*
```


</p>
</details>

<details class="faq box"><summary>Docker Container Operations</summary>
<p>

kubernetes.io bookmark: [docker run](https://kubernetes.io/docs/reference/kubectl/docker-cli-to-kubectl/)

```bash
clear
# Run the docker image
docker run -it --rm -d -p 8080:80 --name my-container my-image:v0.1
```

```bash
clear
# Verify Operation
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
docker image rm my-image:v0.1
```

</p>
</details>

<details class="faq box"><summary>Docker Operations</summary>
<p>

```bash
clear
# Prune all dangling images
docker image prune -a
```

</p>
</details>
<br />

#### 01-02. Name Container Question
* Create a namespace called `pod-namespace`. 
* Create a pod called `pod-1` using `nginx` image. 
* The container in the pod should be named `container-1`.

<details class="faq box"><summary>Prerequisites</summary>
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

##### Help Examples

```bash
clear
# Run the help flag to get examples
kubectl run -h | more
```

Output:

```console
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

kubectl run nginx --image=nginx --dry-run=client 👈👈👈 This example matches most closely to the question. Just needs an output file.

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

<details class="faq box"><summary>Solution</summary>
<p>

kubernetes.io bookmark: [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

```bash
clear
# Using the best example that matches the question
mkdir -p ~/ckad/
kubectl run pod-1 --image=nginx --dry-run=client -o yaml > ~/ckad/01-02.yml
```

```bash
clear
# Edit the YAML file to make required changes
# Use the Question number in case you want to return to the question for reference or for review
vi ~/ckad/01-02.yml
```

```yaml
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
    name: container-1 #👈👈👈 Change from pod-1 to container-1
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
clear
# Apply the YAML file to the Kubernetes API server
kubectl apply -f ~/ckad/01-02.yml
```

```bash
clear
# Quick verification that the pod was created and is working
kubectl get pod --watch
```

</p>
</details>
<br />

#### 01-03. Storage Question
* Create a namespace called `storage-namespace`. 
* Create a Persistent Volume called `my-pv` with `5Gi` storage using hostPath `/mnt/my-host`. 
* Create a Persistent Volume Claim called `my-pvc` with `2Gi` storage. 
* Create a pod called `storage-pod` using the nginx image. 
* Mount the Persistent Volume Claim onto `/my-mount` in `storage-pod`.

<details class="faq box"><summary>Overview</summary>
<p>

![06-pv-pvc-pod](https://user-images.githubusercontent.com/18049790/140639299-1b5a2c1b-139d-4b52-9a35-66d79de5fb71.jpg)

Legend
* PersistentVolume – the low level representation of a storage volume
* PersistentVolumeClaim – the binding between a Pod and PersistentVolume
* Pod – a running container that will consume a PersistentVolume
* StorageClass – allows for dynamic provisioning of PersistentVolumes

[Access Modes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes)
* ReadWriteOnce(RWO) - volume can be mounted as read-write by a single node.
* ReadOnlyMany(ROX) - volume can be mounted read-only by many nodes.
* ReadWriteMany(RWX) - volume can be mounted as read-write by many nodes.
* ReadWriteOncePod(RWOP) - volume can be mounted as read-write by a single Pod.

Notes
* Once a PV is bound to a PVC, that PV is essentially tied to the PVC and cannot be bound to by another PVC. 
* There is a one-to-one mapping of PVs and PVCs. 
* However, multiple pods in the same project can use the same PVC.
* The link between PV and PVC is not explict, instead the PVC makes a some requests for storage. 
* Kubernetes will pick an appropriate PersistentVolume to meet that claim.
* StorageClass provisions PV dynamically, when PVC claims it. 
* StorageClass allows for dynamically provisioned volumes for an incoming claim.


</p>
</details>

<details class="faq box"><summary>Prerequisites</summary>
<p>

```bash
clear
kubectl create namespace storage-namespace
kubectl config set-context --current --namespace=storage-namespace
```

</p>
</details>


<details class="faq box"><summary>Solution - PersistentVolume</summary>
<p>

kubernetes.io bookmark: [Create a PersistentVolume](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolume)

```bash
# Create a YAML file for the PV
mkdir -p ~/ckad/
vi ~/ckad/01-03-pv.yml
```

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv #👈👈👈 Change
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 5Gi #👈👈👈 Change
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/my-host" #👈👈👈 Change
```

```bash
kubectl apply -f ~/ckad/01-03-pv.yml
clear
kubectl get pv
```

Output:

```console
# Note the STATUS=Available
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM
my-pv     5Gi        RWO            Retain           Available
```

</p>
</details>

<details class="faq box"><summary>Solution - PersistentVolumeClaim</summary>
<p>

kubernetes.io bookmark: [Create a PersistentVolumeClaim](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolumeclaim)

```bash
# Create a YAML file for the PVC
mkdir -p ~/ckad/
vi ~/ckad/01-03-pvc.yml
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc #👈👈👈 Change
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi #👈👈👈 Change
```

```bash
kubectl apply -f ~/ckad/01-03-pvc.yml
clear
kubectl get pv
kubectl get pvc
```

Output:

```console
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM
my-pv     5Gi        RWO            Retain           Bound       storage-namespace/my-pvc  # STATUS=Bound means the PV and PVC are linked

NAME     STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
my-pvc   Bound    my-pv    5Gi        RWO            manual         6s                     # STATUS=Bound means the PV and PVC are linked
```

</p>
</details>

<details class="faq box"><summary>Solution - Pod</summary>
<p>

kubernetes.io bookmark: [Create a Pod](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-pod)

```bash
# Create a YAML file for the Pod
mkdir -p ~/ckad/
vi  ~/ckad/01-03-pod.yml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: storage-pod #👈👈👈 Change
spec:
  volumes:
    - name: my-volume
      persistentVolumeClaim:
        claimName: my-pvc #👈👈👈 Change
  containers:
    - name: my-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/my-mount" #👈👈👈 Change
          name: my-volume

```

```bash
kubectl apply -f ~/ckad/01-03-pod.yml
clear
# Verify that the volume is mounted
# Or just kubectl describe pod storage-pod
kubectl describe pod storage-pod | grep -i Mounts -A1
```

Output:

```console
    Mounts:
      /my-mount from my-volume (rw)    # Success
```

</p>
</details>
<br />

#### Clean Up

<details class="faq box"><summary>Clean Up</summary>
<p>

```bash
cd
yes | rm -R ~/ckad/
kubectl delete ns storage-namespace --force
kubectl delete ns pod-namespace --force
kubectl delete pv my-pv
```

</p>
</details>

_End of Section_
