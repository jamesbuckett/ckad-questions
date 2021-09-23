## Sample CKAD Extension Questions and Answers

#### 06-01. List all the Kubernetes resources that can be found inside a namespace. By name only.

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
clear
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

#### 06-02. Give the command to list out all the available API groups on your cluster. Then list out the API's in the `named` group.

<details><summary>show</summary>
<p>

```bash
clear
# Use the kubectl proxy to provide credentials to connect to the API server
# kubectl proxy starts a local proxy service on port 8001
# kubectl proxy uses credentials from kubeconfig file

kubectl proxy &
```

</p>
</details>

<details><summary>show</summary>
<p>

```bash
clear
# List all available API groups from the API server

# /api is called the core API's
# /apis is called the named API's - going forward new features will be made available under this API

# You need to pass the -k or --insecure option to the curl command. 
# This option explicitly allows curl to perform “insecure” SSL connections and transfers.

curl http://localhost:8001 -k | more 
```

Output:
```bash
...
   "/api",
   "/api/v1",
   "/apis",
   "/apis/",
   "/apis/admissionregistration.k8s.io",
   "/apis/admissionregistration.k8s.io/v1",
   "/apis/admissionregistration.k8s.io/v1beta1",
   "/apis/apiextensions.k8s.io",
   "/apis/apiextensions.k8s.io/v1",
   "/apis/apiextensions.k8s.io/v1beta1",
   "/apis/apiregistration.k8s.io",
   "/apis/apiregistration.k8s.io/v1",
   "/apis/apiregistration.k8s.io/v1beta1",
   "/apis/apps",
   "/apis/apps/v1",
   "/apis/authentication.k8s.io",
   "/apis/authentication.k8s.io/v1",
   "/apis/authentication.k8s.io/v1beta1",
   "/apis/authorization.k8s.io",
   "/apis/authorization.k8s.io/v1",
   "/apis/authorization.k8s.io/v1beta1",
   "/apis/autoscaling",
   "/apis/autoscaling/v1",
   "/apis/autoscaling/v2beta1",
   "/apis/autoscaling/v2beta2",
   "/apis/batch",
   "/apis/batch/v1",
   "/apis/batch/v1beta1",
   "/apis/certificates.k8s.io",
   "/apis/certificates.k8s.io/v1",
   "/apis/certificates.k8s.io/v1beta1",
...
```

</p>
</details>

<details><summary>show</summary>
<p>

```bash
clear
# List all supported resource groups under the `named` (apis) group

curl http://locahost:8001/apis -k | grep "name" | more
```

Output:
```bash
...
      "name": "apiregistration.k8s.io",
      "name": "apps",
      "name": "events.k8s.io",
      "name": "authentication.k8s.io",
      "name": "authorization.k8s.io",
      "name": "autoscaling",
      "name": "batch",
      "name": "certificates.k8s.io",
      "name": "networking.k8s.io",
      "name": "extensions",
      "name": "policy",
      "name": "rbac.authorization.k8s.io",
      "name": "storage.k8s.io",
      "name": "admissionregistration.k8s.io",
      "name": "apiextensions.k8s.io",
      "name": "scheduling.k8s.io",
      "name": "coordination.k8s.io",
      "name": "node.k8s.io",
      "name": "discovery.k8s.io",
      "name": "flowcontrol.apiserver.k8s.io",
...
```

</p>
</details>

*End of Section*