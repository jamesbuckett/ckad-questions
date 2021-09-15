## Sample CKAD Networking and Storage Questions and Answers

#### 04-01. Create a namespace called `storage-namespace`. Create a Persistent Volume called `my-pv` with `5Gi` storage using hostPath `/mnt/my-host`. Create a Persistent Volume Claim called `my-pvc` with `2Gi` storage. Create a pod called `storage-pod` using the nginx image. Mount the Persistent Volume Claim onto `/my-mount` in `storage-pod`.

kubernetes.io: [Create a PersistentVolume](Create a PersistentVolume)

<details><summary>show</summary>
<p>

```bash
kubectl create namespace storage-namespace
kubectl config set-context --current --namespace=storage-namespace
```

```bash
vi 04-01-pv.yml
```

kubernetes.io: [Create a PersistentVolume](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolume)

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
kubectl apply -f 04-01-pv.yml
kubectl get pv
```

</p>
</details>

<details><summary>show</summary>
<p>

```bash
vi 04-01-pvc.yml
```

kubernetes.io: [Create a PersistentVolumeClaim](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolumeclaim)

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
kubectl apply -f 04-02-pvc.yml
kubectl get pv
kubectl get pvc
```

</p>
</details>

<details><summary>show</summary>
<p>

```bash
vi 04-01-pod.yml
```

kubernetes.io: [Create a Pod](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-pod)

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
kubectl apply -f 04-02-pod.yml
# Verify that the volume is mounted
kubectl describe pod storage-pod | grep -i A2 volume
```

</p>
</details>


#### 04-02. Sample Question.

<details><summary>show</summary>
<p>

```bash
Sample

```

</p>
</details>

*End of Section*