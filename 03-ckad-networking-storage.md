## Sample CKAD Networking and Storage Questions and Answers

#### 04-01. Create a namespace called `storage-namespace`. Create a Persistent Volume called `my-pv` with `5Gi` storage using hostPath `/mnt/my-host`. Create a Persistent Volume Claim called `my-pvc` with `2Gi` storage. Create a pod called `storage-pod` using the nginx image. Mount the Persistent Volume Claim onto `/my-mount` in `storage-pod`.

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
vi 04-01-pv.yml
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
kubectl apply -f 04-01-pv.yml
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
vi 04-01-pvc.yml
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
kubectl apply -f 04-01-pvc.yml
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
vi 04-01-pod.yml
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
kubectl apply -f 04-01-pod.yml
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


#### 04-02. Create a namespace called `service-namespace`. Create a pod called `service-pod` using the `nginx` image and exposing port `80`. Label the pod `tier=web`. Create a service for the pod called `my-service` allowing for communication inside the cluster. Let the service expose port 8080. Create an ingress called `my-ingress` to expose the service outside the cluster.

<details><summary>show</summary>
<p>

```bash
kubectl create namespace service-namespace
kubectl config set-context --current --namespace=service-namespace
```

```bash
kubectl run -h | more
```

Output:
```bash
Examples:
  # Start a nginx pod
  kubectl run nginx --image=nginx
  
  # Start a hazelcast pod and let the container expose port 5701
  kubectl run hazelcast --image=hazelcast/hazelcast --port=5701 ### This example matches most closely to the question.
  
  # Start a hazelcast pod and set environment variables "DNS_DOMAIN=cluster" and "POD_NAMESPACE=default" in the container
  kubectl run hazelcast --image=hazelcast/hazelcast --env="DNS_DOMAIN=cluster" --env="POD_NAMESPACE=default"
  
  # Start a hazelcast pod and set labels "app=hazelcast" and "env=prod" in the container
  kubectl run hazelcast --image=hazelcast/hazelcast --labels="app=hazelcast,env=prod" ### This example matches most closely to the question.
  
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
kubectl run service-pod --image=nginx --port=80  --labels="tier=web"
kubectl get all
```

</p>
</details>


<details><summary>show</summary>
<p>

```bash
kubectl expose -h | more
```
Output:
```bash
Examples:
  # Create a service for a replicated nginx, which serves on port 80 and connects to the containers on port 8000
  kubectl expose rc nginx --port=80 --target-port=8000
  
  # Create a service for a replication controller identified by type and name specified in "nginx-controller.yaml",
which serves on port 80 and connects to the containers on port 8000
  kubectl expose -f nginx-controller.yaml --port=80 --target-port=8000 
  
  # Create a service for a pod valid-pod, which serves on port 444 with the name "frontend"
  kubectl expose pod valid-pod --port=444 --name=frontend  ### This example matches most closely to the question.
    
  # Create a second service based on the above service, exposing the container port 8443 as port 443 with the name
"nginx-https"
  kubectl expose service nginx --port=443 --target-port=8443 --name=nginx-https
  
  # Create a service for a replicated streaming application on port 4100 balancing UDP traffic and named 'video-stream'.
  kubectl expose rc streamer --port=4100 --protocol=UDP --name=video-stream
  
  # Create a service for a replicated nginx using replica set, which serves on port 80 and connects to the containers on
port 8000
  kubectl expose rs nginx --port=80 --target-port=8000 ### This example matches most closely to the question.
  
  # Create a service for an nginx deployment, which serves on port 80 and connects to the containers on port 8000
  kubectl expose deployment nginx --port=80 --target-port=8000
```

```bash
kubectl expose pod service-pod --port=8080 --target-port=80 --name=my-service
kubectl get all
kubectl get ep
```

</p>
</details>

<details><summary>show</summary>
<p>

*This part is under development until the new curriculum is released*

kubernetes.io [The Ingress resource](https://kubernetes.io/docs/concepts/services-networking/ingress/#the-ingress-resource)

```bash
vi q03-02-ing.yml
```

```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress          # Change
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /              # Change
        pathType: Prefix
        backend:
          service:
            name: my-service # Change
            port:
              number: 8080   # Change
```

```bash
kubectl apply -f q03-02-ing.yml

# Pod Address under IP heading
kubectl get pod -o wide 

# The Pod is an endpoint listed under ENDPOINTS with port :80
kubectl get ep 

# The Service IP listed under CLUSTER-IP with PORT(S) :8080
kubectl get service -o wide

# Ingress IP listed under ADDRESS`
kubectl get ingress
```

```bash
NAME         CLASS    HOSTS   ADDRESS           PORTS   AGE
my-ingress   <none>   *       144.126.242.138   80      4m34s
```

```bash
curl localhost
```


Output: 
```bash
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

</p>
</details>


#### Clean Up 

```bash
kubectl delete ns storage-namespace --force
kubectl delete ns service-namespace --force
```

*End of Section*