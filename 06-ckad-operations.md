## Sample CKAD Operational Questions and Answers

#### 06-01. First list all the pods in the cluster by CPU consumption. Then list all the pods in the cluster by Memory consumption.

<details><summary>show</summary>
<p>

By default the metrics server required for the `kubectl top` command is not present on Docker Desktop.

Install the [metrics server](https://github.com/kubernetes-sigs/metrics-server) with the following command:

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

```bash
kubectl top pods -A --sort-by=cpu | more
```

Output:
```bash
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
kubectl top pods -A --sort-by=memory | more
```

Output:
```bash
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

#### 06-02. Create a pod called `json-pod` using image `nginx` in namespace `json-namespace`. Create the namespace. Obtain the `hostIP` address using `JSONPath`.

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
```

Output:
```bash

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
    "status": {                          ## First element: .status
        "conditions": [
            {
                "lastProbeTime": null,
                "lastTransitionTime": "2021-09-11T07:39:32Z",
                "status": "True",
                "type": "Initialized"
            },
...
        "hostIP": "10.130.0.5",         ## Second Element: .status.hostIP 
        "phase": "Running",
        "podIP": "10.244.2.198",

# So the JSON path is .status.hostIP
```

</p>
</details>

<details><summary>show</summary>
<p>

```bash
# Another way to get the JSONPath
kubectl explain pod.status
# kubectl explain pod.status --recursive

Output:
```bash
KIND:     Pod
VERSION:  v1

RESOURCE: status <Object>            ## First element: .status

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

   hostIP       <string>               ## Second element: .status.hostIP
     IP address of the host to which the pod is assigned. Empty if not yet
     scheduled.
```

kubernetes.io:[JSONPath Support](https://kubernetes.io/docs/reference/kubectl/jsonpath/)
```bash
kubectl get pod json-pod -o jsonpath={.status.hostIP}    
```

</p>
</details>

#### 06-03. Output all the events for all namespaces by creation date.

<details><summary>show</summary>
<p>

kubernetes.io: [Viewing, finding resources](https://kubernetes.io/docs/reference/kubectl/cheatsheet/#viewing-finding-resources)

```bash
kubectl get events -A --sort-by=.metadata.creationTimestamp
```

</p>
</details>

#### 06-04. Create a pod called `log-pod` using image `nginx` in namespace `log-namespace`. Create the namespace. Obtain the `logs` for the nginx pod for the `last hour`.

<details><summary>show</summary>
<p>

```bash
kubectl create namespace log-namespace
kubectl run log-pod --image=nginx -n log-namespace
kubectl config set-context --current --namespace=log-namespace
```

</p>
</details>

<details><summary>show</summary>
<p>

```bash
kubectl logs -h | more
```

Output:
```bash
Examples:
  # Return snapshot logs from pod nginx with only one container
  kubectl logs nginx

  # Return snapshot logs from pod nginx with multi containers
  kubectl logs nginx --all-containers=true

  # Return snapshot logs from all containers in pods defined by label app=nginx
  kubectl logs -l app=nginx --all-containers=true

  # Return snapshot of previous terminated ruby container logs from pod web-1
  kubectl logs -p -c ruby web-1

  # Begin streaming the logs of the ruby container in pod web-1
  kubectl logs -f -c ruby web-1

  # Begin streaming the logs from all containers in pods defined by label app=nginx
  kubectl logs -f -l app=nginx --all-containers=true

  # Display only the most recent 20 lines of output in pod nginx
  kubectl logs --tail=20 nginx

  # Show all logs from pod nginx written in the last hour
  kubectl logs --since=1h nginx

  # Show logs from a kubelet with an expired serving certificate
  kubectl logs --insecure-skip-tls-verify-backend nginx

  # Return snapshot logs from first container of a job named hello
  kubectl logs job/hello

  # Return snapshot logs from container nginx-1 of a deployment named nginx
  kubectl logs deployment/nginx -c nginx-1
```

```bash
# Straight forward match in the examples
kubectl logs --since=1h nginx
```

</p>
</details>

*End of Section*