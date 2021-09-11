## Sample CKAD Questions and Answers

#### 01. List all the kubernetes resources that can be found in a namespace. By name only.

<details><summary>show</summary>
<p>

```bash
kubectl api-resources --namespaced=true   # From Kubernetes.io Bookmarks..Namespace

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
kubectl api-resources --namespaced=true -o name

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

</p>
</details>

<details><summary>show</summary>
<p>

```bash
# Using the best example that matches the question
# --dry-run=client -o yaml from Kubernetes.io..Cheat Sheet
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

</p>
</details>

<details><summary>show</summary>
<p>

```bash
# Using the best example that matches the question
kubectl create deployment my-deployment --image=nginx --repliacs=3 -n my-namespace --dry-run=client -o yaml > q3.yml
```

```bash
# Edit the YAML file to make required changes
vi q3.yml
```

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
        resources:                     # From Kubernetes.io Bookmarks..Core Pod..Pod-Limits and Requests
          requests:
            memory: "25Mi"
          limits:
            memory: "100Mi"
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
kubectl expose -h

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

NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
my-deployment   ClusterIP   10.245.79.74   <none>        80/TCP    103s
```

```bash
# A quicker check is to see if the Pod Endpoints are being load balanced
kubectl get endpoints

NAME            ENDPOINTS                                         AGE
my-deployment   10.244.0.250:80,10.244.1.132:80,10.244.1.246:80   5m20s
# The three replicas internal endpoints are registered
```

</p>
</details>

#### 05. First list all the pods in the cluster by CPU consumption. Then list all the pods in the cluster by Memory consumption.

<details><summary>show</summary>
<p>

```bash
kubectl top pods -A --sort-by=cpu

NAMESPACE                 NAME                                                    CPU(cores)   MEMORY(bytes)
default                   falco-pxf8g                                             51m          55Mi
ns-loki                   loki-release-prometheus-server-6d4f4df478-9z2f8         38m          356Mi
ns-demo                   adservice-68444cb46c-jvc86                              23m          202Mi
ns-loki                   loki-release-promtail-prvvn                             13m          34Mi
ns-demo                   recommendationservice-b4cf8f489-xwv49                   13m          69Mi
...
```

</p>
</details>

<details><summary>show</summary>
<p>

```bash
kubectl top pods -A --sort-by=memory

NAMESPACE                 NAME                                                    CPU(cores)   MEMORY(bytes)
ns-loki                   loki-release-prometheus-server-6d4f4df478-9z2f8         11m          356Mi
ns-demo                   adservice-68444cb46c-jvc86                              20m          202Mi
kube-system               cilium-gcnbl                                            6m           165Mi
kube-system               cilium-htrth                                            18m          163Mi
kube-system               cilium-8h6vd                                            5m           162Mi
kube-system               cilium-ml27n                                            11m          161Mi
...
```

</p>
</details>

#### 06. Create a pod called json-pod using image nginx in namespace json-namespace. Create the namespace. Obtain the hostIP address using JSONPath.

<details><summary>show</summary>
<p>

```bash
kubectl create namespace json-namespace
kubectl run json-pod --image=nginx -n json-namespace
kubectl config set-context --current --namespace=json-namespace
```

</p>
</details>

<details><summary>show</summary>
<p>

```bash
kubectl get pod json-pod -o json

{
    "apiVersion": "v1",
    "kind": "Pod",
    "metadata": {
        "creationTimestamp": "2021-09-11T07:39:32Z",
        "labels": {
            "run": "json-pod"
        },
        "name": "json-pod",
        "namespace": "json-namespace",
        "resourceVersion": "8441827",
        "uid": "f2e7c606-aeb5-4036-b542-33b573f41008"
    },
    "spec": {
        "containers": [
            {
                "image": "nginx",
                "imagePullPolicy": "Always",
                "name": "json-pod",
                "resources": {},
                "terminationMessagePath": "/dev/termination-log",
                "terminationMessagePolicy": "File",
                "volumeMounts": [
                    {
                        "mountPath": "/var/run/secrets/kubernetes.io/serviceaccount",
                        "name": "kube-api-access-j27qd",
                        "readOnly": true
                    }
                ]
            }
        ],
        "dnsPolicy": "ClusterFirst",
        "enableServiceLinks": true,
        "nodeName": "digital-ocean-pool-80e4x",
        "preemptionPolicy": "PreemptLowerPriority",
        "priority": 0,
        "restartPolicy": "Always",
        "schedulerName": "default-scheduler",
        "securityContext": {},
        "serviceAccount": "default",
        "serviceAccountName": "default",
        "terminationGracePeriodSeconds": 30,
        "tolerations": [
            {
                "effect": "NoExecute",
                "key": "node.kubernetes.io/not-ready",
                "operator": "Exists",
                "tolerationSeconds": 300
            },
            {
                "effect": "NoExecute",
                "key": "node.kubernetes.io/unreachable",
                "operator": "Exists",
                "tolerationSeconds": 300
            }
        ],
        "volumes": [
            {
                "name": "kube-api-access-j27qd",
                "projected": {
                    "defaultMode": 420,
                    "sources": [
                        {
                            "serviceAccountToken": {
                                "expirationSeconds": 3607,
                                "path": "token"
                            }
                        },
                        {
                            "configMap": {
                                "items": [
                                    {
                                        "key": "ca.crt",
                                        "path": "ca.crt"
                                    }
                                ],
                                "name": "kube-root-ca.crt"
                            }
                        },
                        {
                            "downwardAPI": {
                                "items": [
                                    {
                                        "fieldRef": {
                                            "apiVersion": "v1",
                                            "fieldPath": "metadata.namespace"
                                        },
                                        "path": "namespace"
                                    }
                                ]
                            }
                        }
                    ]
                }
            }
        ]
    },
    "status": {
        "conditions": [
            {
                "lastProbeTime": null,
                "lastTransitionTime": "2021-09-11T07:39:32Z",
                "status": "True",
                "type": "Initialized"
            },
            {
                "lastProbeTime": null,
                "lastTransitionTime": "2021-09-11T07:39:43Z",
                "status": "True",
                "type": "Ready"
            },
            {
                "lastProbeTime": null,
                "lastTransitionTime": "2021-09-11T07:39:43Z",
                "status": "True",
                "type": "ContainersReady"
            },
            {
                "lastProbeTime": null,
                "lastTransitionTime": "2021-09-11T07:39:32Z",
                "status": "True",
                "type": "PodScheduled"
            }
        ],
        "containerStatuses": [
            {
                "containerID": "containerd://3bb7b9561dc70787bb765165e7ed58be1b4837554be4b0b35ea5967fc86c1c35",
                "image": "docker.io/library/nginx:latest",
                "imageID": "docker.io/library/nginx@sha256:853b221d3341add7aaadf5f81dd088ea943ab9c918766e295321294b035f3f3e",
                "lastState": {},
                "name": "json-pod",
                "ready": true,
                "restartCount": 0,
                "started": true,
                "state": {
                    "running": {
                        "startedAt": "2021-09-11T07:39:42Z"
                    }
                }
            }
        ],
        "hostIP": "10.130.0.5",             # This is what we are looking for
        "phase": "Running",
        "podIP": "10.244.2.198",
        "podIPs": [
            {
                "ip": "10.244.2.198"
            }
        ],
        "qosClass": "BestEffort",
        "startTime": "2021-09-11T07:39:32Z"
    }
}
```

```bash
# Reduced output to walk back to JSON root:
    "status": {                                             ## First element
        "conditions": [
            {
                "lastProbeTime": null,
                "lastTransitionTime": "2021-09-11T07:39:32Z",
                "status": "True",
                "type": "Initialized"
            },
...
        "hostIP": "10.130.0.5",               ## Second Element
        "phase": "Running",
        "podIP": "10.244.2.198",

# So the JSON path is .status.hostIP
```

```bash
# Another way to get the JSONPath
kubectl explain pod.status
kubectl explain pod.status --recursive

KIND:     Pod
VERSION:  v1

RESOURCE: status <Object>            ## First element

DESCRIPTION:
     Most recently observed status of the pod. This data may not be up to date.
     Populated by the system. Read-only. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

     PodStatus represents information about the status of a pod. Status may
     trail the actual state of a system, especially if the node that hosts the
     pod cannot contact the control plane.

FIELDS:
   conditions   <[]Object>
     Current service state of pod. More info:
     https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle#pod-conditions

   containerStatuses    <[]Object>
     The list has one entry per container in the manifest. Each entry is
     currently the output of `docker inspect`. More info:
     https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle#pod-and-container-status

   ephemeralContainerStatuses   <[]Object>
     Status for any ephemeral containers that have run in this pod. This field
     is alpha-level and is only populated by servers that enable the
     EphemeralContainers feature.

   hostIP       <string>                                          ## Second element
     IP address of the host to which the pod is assigned. Empty if not yet
     scheduled.
```

```bash
kubectl get pod json-pod -o jsonpath={.status.hostIP}    ## From Kubernetes.io Bookmarks..Operations..
10.130.0.5
```

</p>
</details>
