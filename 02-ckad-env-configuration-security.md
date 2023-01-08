## Sample CKAD Environment, Configuration and Security Q&A

### Application Environment, Configuration and Security – 25%

- Discover and use resources that extend Kubernetes (CRD) [\*\*](https://github.com/jamesbuckett/ckad-questions/blob/main/02-ckad-env-configuration-security.md#02-01-list-all-the-custom-resource-definitions-installed-in-a-cluster-calico-is-a-crd-list-out-how-to-obtain-the-correct-resource-name-to-query-a-calico-network-policy-and-not-the-default-kubernetes-network-policy)
- Understand authentication, authorization and admission control
- Understanding and defining resource requirements, limits and quotas [\*\*](https://github.com/jamesbuckett/ckad-questions/blob/main/02-ckad-env-configuration-security.md#02-02-create-a-namespace-called-quota-namespace-create-a-resource-quota-for-this-namespace-called-my-quota-set-a-memory-reservation-of-2gi-set-a-cpu-reservation-of-500mi)
- Understand ConfigMaps
- Create & consume Secrets [\*\*](https://github.com/jamesbuckett/ckad-questions/blob/main/02-ckad-env-configuration-security.md#02-02-create-a-namespace-called-quota-namespace-create-a-resource-quota-for-this-namespace-called-my-quota-set-a-memory-reservation-of-2gi-set-a-cpu-reservation-of-500mi)
- Understand ServiceAccounts [\*\*](https://github.com/jamesbuckett/ckad-questions/blob/main/02-ckad-env-configuration-security.md#02-04-create-a-namespace-called-serviceaccount-namespace-create-a-pod-called-serviceaccount-pod-using-nginx-image-create-a-seviceaccount-called-my-serviceaccount-update-the-pod-to-use-the-new-serviceaccount-display-the-token-for-the-new-serviceaccount)
- Understand SecurityContexts
<br />

#### 02-01. Custom Resource Definition Question
* List all the Custom Resource Definitions installed in a cluster. 
* Calico is a CRD. 
* List out how to obtain the correct resource name to query a Calico Network Policy and not the default Kubernetes Network Policy.

<details class="faq box"><summary>Overview</summary>
<p>

```bash
clear
# kubectl get Custom Resource Definitions
kubectl get crds
```

Output:

```console
NAME                                                  CREATED AT
bgpconfigurations.crd.projectcalico.org               2021-09-24T05:26:26Z
bgppeers.crd.projectcalico.org                        2021-09-24T05:26:26Z
blockaffinities.crd.projectcalico.org                 2021-09-24T05:26:26Z
clusterinformations.crd.projectcalico.org             2021-09-24T05:26:26Z
extensionservices.projectcontour.io                   2021-09-24T05:26:16Z
felixconfigurations.crd.projectcalico.org             2021-09-24T05:26:26Z
globalnetworkpolicies.crd.projectcalico.org           2021-09-24T05:26:26Z
globalnetworksets.crd.projectcalico.org               2021-09-24T05:26:26Z
hostendpoints.crd.projectcalico.org                   2021-09-24T05:26:26Z
httpproxies.projectcontour.io                         2021-09-24T05:26:16Z
ipamblocks.crd.projectcalico.org                      2021-09-24T05:26:26Z
ipamconfigs.crd.projectcalico.org                     2021-09-24T05:26:26Z
ipamhandles.crd.projectcalico.org                     2021-09-24T05:26:26Z
ippools.crd.projectcalico.org                         2021-09-24T05:26:26Z
kubecontrollersconfigurations.crd.projectcalico.org   2021-09-24T05:26:26Z
networkpolicies.crd.projectcalico.org                 2021-09-24T05:26:26Z
networksets.crd.projectcalico.org                     2021-09-24T05:26:26Z
tlscertificatedelegations.projectcontour.io           2021-09-24T05:26:16Z
```

</p>
</details>

<details class="faq box"><summary>Solution</summary>
<p>

```bash
kubectl config set-context --current --namespace=default
clear
kubectl api-resources -o name | grep calico
```

Output:

```console
bgpconfigurations.crd.projectcalico.org
bgppeers.crd.projectcalico.org
blockaffinities.crd.projectcalico.org
clusterinformations.crd.projectcalico.org
felixconfigurations.crd.projectcalico.org
globalnetworkpolicies.crd.projectcalico.org
globalnetworksets.crd.projectcalico.org
hostendpoints.crd.projectcalico.org
ipamblocks.crd.projectcalico.org
ipamconfigs.crd.projectcalico.org
ipamhandles.crd.projectcalico.org
ippools.crd.projectcalico.org
kubecontrollersconfigurations.crd.projectcalico.org
networkpolicies.crd.projectcalico.org 👈👈👈 This is the Calico Resource Type that we want
networksets.crd.projectcalico.org
```

```bash
clear
# This is the command to get all Calico Network Policies
# If you run this command it will return: "No resources found in storage-namespace namespace"
# As we have not created any Calico Network Policies
kubectl get networkpolicies.crd.projectcalico.org
```

</p>
</details>
<br />

#### 02-02. Quota and LimitRange Question
* Create a namespace called `quota-namespace`. 
* Create a Resource Quota for this namespace called `my-quota`. 
* Set a hard memory reservation of `2Gi`. 
* Set a hard CPU reservation of `500m`. 
* Create a LimitRange for this namespace called `my-limit` which limits Pods to to a maximum of `1Gi` memory and `250m` CPU.

<details class="faq box"><summary>Prerequisites</summary>
<p>

```bash
clear
kubectl create namespace quota-namespace
kubectl config set-context --current --namespace=quota-namespace
```

</p>
</details>

<details class="faq box"><summary>Help</summary>
<p>

```bash
clear
kubectl create quota -h | more
```

Output

```console
  # Create a new resource quota named my-quota
  kubectl create quota my-quota --hard=cpu=1,memory=1G,pods=2,services=3,replicationcontrollers=2,resourcequotas=1,secrets=5,persistentvolumeclaims=10 👈👈👈 This example matches most closely to the question.

  # Create a new resource quota named best-effort
  kubectl create quota best-effort --hard=pods=100 --scopes=BestEffort
```

</p>
</details>

<details class="faq box"><summary>Solution</summary>
<p>

```bash
clear
kubectl create quota my-quota --hard=cpu=500m,memory=2G
kubectl get quota
```

Output:

```console
NAME       AGE    REQUEST                      LIMIT
my-quota   118s   cpu: 0/500m, memory: 0/2G
```

```bash
# Try to run a Pod with resource requests exceeding the quota
mkdir -p ~/ckad/
clear
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml | kubectl set resources -f - --requests=cpu=1000m,memory=4Gi --limits=cpu=1000m,memory=4Gi --local -o yaml > ~/ckad/02-02-exceed.yml
kubectl apply -f ~/ckad/02-02-exceed.yml
```

In English:

| Value   | Translation       |
| ------- | ----------------- |
| REQUEST | Minimum (Request) |
| LIMIT   | Maximum (Limits)  |


LimitRange
* The LimitRange object cannot be created from the command line
* Use the [Configure LimitRange](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-constraint-namespace/#create-a-limitrange-and-a-pod) bookmark to obtain a code snippet

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: my-limit
spec:
  limits:
  - max:
      memory: 1Gi  # 👈👈👈 Change this value 
      cpu: 250m    # 👈👈👈 Change this value  
    type: Pod      # 👈👈👈 Change this value  
```


```bash
# This Pod is within the resource requests of the Resource Quota and LimitRange 
mkdir -p ~/ckad/
clear
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml | kubectl set resources -f - --requests=cpu=200m,memory=512Mi --limits=cpu=200m,memory=512Mi --local -o yaml > ~/ckad/02-02-succeed.yml
kubectl apply -f ~/ckad/02-02-succeed.yml
kubectl get all
kubectl get quota
```

Output:

```console
NAME       AGE   REQUEST                           LIMIT
my-quota   19m   cpu: 250m/500m, memory: 1Gi/2G
```

[Meaning of memory](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#meaning-of-memory)

- Limits and requests for MEMORY are measured in bytes.
- You can express memory as a plain integer or as a fixed-point number using one of these suffixes: E, P, T, G, M, k.
- You can also use the power-of-two equivalents: Ei, Pi, Ti, Gi, Mi, Ki.

In English:

| Suffix | Description |
| ------ | ----------- |
| E/Ei   | Exabyte     |
| P/Pi   | Petabyte    |
| T/Ti   | Terrabyte   |
| G/Gi   | Gigabyte    |
| M/Mi   | Megabyte    |
| k/Ki   | Kilobyte    |

</p>
</details>
<br />

#### 02-03. Secret Question
* Create a namespace called `secret-namespace`. 
* Create a secret in this namespace called `my-secret`. 
* The secret should be immutable and contain the literal values `user=bob` and `password=123456`. 
* Create a pod called called `secret-pod` using the `nginx` image. 
* The pod should consume the secret as environmental variables `SECRET-ENV-USER` and `SECRET-ENV-PASSWORD`.

<details class="faq box"><summary>Prerequisites</summary>
<p>

```bash
clear
kubectl create namespace secret-namespace
kubectl config set-context --current --namespace=secret-namespace
```

</p>
</details>

<details class="faq box"><summary>Help</summary> 
<p>

##### Help Examples

Three types of secret:

- generic
- docker-registry
- tls

##### Help Examples - GENERIC

Create a secret from a local file, directory, or literal value

```bash
clear
# kubectl create secret -h
kubectl create secret generic -h | more
```

Output:

```console
Examples:
  # Create a new secret named my-secret with keys for each file in folder bar
  kubectl create secret generic my-secret --from-file=path/to/bar

  # Create a new secret named my-secret with specified keys instead of names on disk
  kubectl create secret generic my-secret --from-file=ssh-privatekey=path/to/id_rsa
--from-file=ssh-publickey=path/to/id_rsa.pub

  # Create a new secret named my-secret with key1=supersecret and key2=topsecret
  kubectl create secret generic my-secret --from-literal=key1=supersecret --from-literal=key2=topsecret 👈👈👈 This example matches most closely to the question.

  # Create a new secret named my-secret using a combination of a file and a literal
  kubectl create secret generic my-secret --from-file=ssh-privatekey=path/to/id_rsa --from-literal=passphrase=topsecret

  # Create a new secret named my-secret from an env file
  kubectl create secret generic my-secret --from-env-file=path/to/bar.env
```

##### Help Examples - TLS (Transport Layer Security)

Create a TLS secret

```bash
clear
# kubectl create secret -h
kubectl create secret tls -h | more
```

```console
Examples:
  # Create a new TLS secret named tls-secret with the given key pair
  kubectl create secret tls tls-secret --cert=path/to/tls.cert --key=path/to/tls.key
```

</p>
</details>

<details class="faq box"><summary>Solution</summary>
<p>

```bash
clear
# Create a generic secret
mkdir -p ~/ckad/
kubectl create secret generic my-secret --from-literal=user=bob --from-literal=password=123456 --dry-run=client -o yaml > ~/ckad/02-03-secret.yml
vi ~/ckad/02-03-secret.yml
```

kubernetes.io bookmark: [Immutable Secrets](https://kubernetes.io/docs/concepts/configuration/secret/#secret-immutable)

```yaml
apiVersion: v1
data:
  password: MTIzNDU2
  user: Ym9i
immutable: true   #👈👈👈 From Immutable Secrets link above
kind: Secret
metadata:
  creationTimestamp: null
  name: my-secret
```

[my-secret](examples/02-03-secret.yml)

```bash
clear
# Apply the YAML file to the Kubernetes API server
# The secret is available to all pods in the namespace
kubectl apply -f ~/ckad/02-03-secret.yml
clear
# Verify that the secret got created
kubectl get secret my-secret
kubectl describe secret my-secret
```

```bash
clear
# Now to create the pod that will consume the secret
kubectl run secret-pod --image=nginx --restart=Never -n secret-namespace --dry-run=client -o yaml > ~/ckad/02-03-pod.yml
vi ~/ckad/02-03-pod.yml
```

kubernetes.io bookmark: [Using Secrets as environment variables](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-environment-variables)

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: secret-pod
  name: secret-pod
  namespace: secret-namespace
spec:
  containers:
  - image: nginx
    name: secret-pod
    env:                             #👈👈👈 From Using Secrets as environment variables link above
      - name: SECRET-ENV-USER        #👈👈👈 From Using Secrets as environment variables link above
        valueFrom:                   #👈👈👈 From Using Secrets as environment variables link above
          secretKeyRef:              #👈👈👈 From Using Secrets as environment variables link above
            name: my-secret          #👈👈👈 From Using Secrets as environment variables link above
            key: user                #👈👈👈 From Using Secrets as environment variables link above
      - name: SECRET-ENV-PASSWORD    #👈👈👈 From Using Secrets as environment variables link above
        valueFrom:                   #👈👈👈 From Using Secrets as environment variables link above
          secretKeyRef:              #👈👈👈 From Using Secrets as environment variables link above
            name: my-secret          #👈👈👈 From Using Secrets as environment variables link above
            key: password            #👈👈👈 From Using Secrets as environment variables link above
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
clear
# Apply the YAML file to the Kubernetes API server
kubectl apply -f ~/ckad/02-03-pod.yml
```

```bash
clear
# Quick verification that the deployment was created and the secret is visible as an environmental variable
kubectl exec secret-pod -- env
```

Output:

```console
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=secret-pod
NGINX_VERSION=1.21.3
NJS_VERSION=0.6.2
PKG_RELEASE=1~buster
SECRET-ENV-USER=bob #👈👈👈 Success
SECRET-ENV-PASSWORD=123456 #👈👈👈 Success
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.245.0.1
KUBERNETES_SERVICE_HOST=10.245.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.245.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.245.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
TERM=xterm
HOME=/root
```

</p>
</details>
<br />

#### 02-04. ServiceAccount Question
* Create a namespace called `serviceaccount-namespace`. 
* Create a pod called `serviceaccount-pod` using `nginx` image. 
* Create a SeviceAccount called: `my-serviceaccount`. 
* Update the pod to use the new ServiceAccount. 
* Display the token for the new ServiceAccount.

<details class="faq box"><summary>Prerequisites</summary>
<p>

```bash
clear
kubectl create namespace serviceaccount-namespace
kubectl config set-context --current --namespace=serviceaccount-namespace
```

</p>
</details>

<details class="faq box"><summary>Solution - Create the serviceAccount and display the token</summary>
<p>

```bash
clear
# Create the serviceAccount
kubectl create sa my-serviceaccount
```

```bash
# Get the corresponding secret created for the new serviceAccount
clear
kubectl get secret
```

```bash
# Get the token out of the secret
clear
kubectl describe secret my-serviceaccount-token-***** #👈👈👈 Replace ***** with your values
```

Output:

```console
Name:         my-serviceaccount-token-nptmw
Namespace:    serviceaccount-namespace
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: my-serviceaccount
              kubernetes.io/service-account.uid: c8e68650-5fcb-4654-a8f7-99fedaba4356

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1156 bytes
namespace:  24 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IkFEcU0xS1hKTTQtZFR6bjl3UHlJZ09rNWpobDkyTmxBV05GSmFvZnpjY2sifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJzZXJ2aWNlYWNjb3VudC1uYW1lc3BhY2UiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlY3JldC5uYW1lIjoibXktc2VydmljZWFjY291bnQtdG9rZW4tbnB0bXciLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoibXktc2VydmljZWFjY291bnQiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJjOGU2ODY1MC01ZmNiLTQ2NTQtYThmNy05OWZlZGFiYTQzNTYiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6c2VydmljZWFjY291bnQtbmFtZXNwYWNlOm15LXNlcnZpY2VhY2NvdW50In0.2R9cp-NzmtTzgJFiMkU1e-UdhoH5pa1cUPZULjNvJzwxrY7jRRwlIlAfAMcUf5dYdVeDpVla8oIRR_p_6R-SyoP6QbzxVtUWU8ApgYn_daH6lRFFtjv3-t9cOBWzZeBXdnmLw2u6t8dgjZ-dh7ExIgRfYrJQ_E_m3B1GNl-XpRC2xQ_-zXMOyHbhs1_Tx3aL5sBzWxmaR_I7X-9S--66gVVjuXEZwooZQblX3vjv3xWrfMDQb0bNjuFe7SK9FpFeLCPFd_yqIfKfwVhNogubhXiSWSoN_MxlUCmD5dxjkVJNrdhQJ5NHhI9-tzpa3cqsUyL0pjm7OexF-woYp3p96g #👈👈👈 This is the token that can be used to authenticate to the API Server
```

</p>
</details>

<details class="faq box"><summary>Solution - Create the pod and update the pod to use the new serviceAccount</summary>
<p>

```bash
# Create the pod declaratively
mkdir -p ~/ckad/
kubectl run serviceaccount-pod --image=nginx --dry-run=client -o yaml > ~/ckad/02-04.yml
```

```bash
# Edit the manifest file
vi ~/ckad/02-04.yml
```

kubernetes.io bookmark: [Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: serviceaccount-pod
  name: serviceaccount-pod
spec:
  serviceAccountName: my-serviceaccount #👈👈👈 Configure Service Accounts for Pods, catch the value is serviceAccountName and NOT serviceaccount
  containers:
  - image: nginx
    name: serviceaccount-pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
# Apply the YAML file to the Kubernetes API server
kubectl apply -f ~/ckad/02-04.yml
```

```bash
# Bonus Section # Verify your work by checking the serviceAccount in use via JSONPath
kubectl get pod serviceaccount-pod -o jsonpath={.spec.serviceAccountName}
```

Output

```console
my-serviceaccount
```

</p>
</details>
<br />

#### 02-05. RBAC Question
* "Error from server (Forbidden): pod is forbidden: User `rbac-sa` cannot `delete` resource `pods` in API group `apps` in the namespace `rbac-namespace`" 
* Fix the problem.

<details class="faq box"><summary>Overview</summary>
<p>

![rbac](https://user-images.githubusercontent.com/18049790/145154335-3086bbce-14d9-4a7d-83b7-92a61d4a9456.jpg)

</p>
</details>

<details class="faq box"><summary>RBAC Combinations</summary>
<p>

In English:

| Combination | Allowed/Disallowed |
| ------ | ----------- |
|Role + RoleBinding| Allowed     |
|ClusterRole + ClusterRoleBinding   | Allowed    |
| ClusterRole + RoleBinding   | Allowed   |
| Role + ClusterRoleBinding   | DISALLOWED    |

</p>
</details>

<details class="faq box"><summary>Prerequisites</summary>
<p>

```bash
clear
kubectl create namespace rbac-namespace #👈👈👈 Create a namespace
kubectl config set-context --current --namespace=rbac-namespace #👈👈👈Change directory into the namespace
kubectl create sa rbac-sa #👈👈👈 Create a Service Account (Who)
kubectl create deployment rbac-deployment --image=nginx --replicas=3 #👈👈👈 Create a deployment 
kubectl create role rbac-role --verb=get,watch --resource=pods,pods/status #👈👈👈Create a Role (What and Where)
kubectl create rolebinding rbac-rolebinding --role=rbac-role --serviceaccount=rbac-namespace:rbac-sa #👈👈👈 Bind Account and Role
```

```bash
kubectl auth can-i get pods --as=system:serviceaccount:rbac-namespace:rbac-sa # yes to get verb
kubectl auth can-i delete pods --as=system:serviceaccount:rbac-namespace:rbac-sa # no to delete verb
```

</p>
</details>

<details class="faq box"><summary>Solution</summary>
<p>

```bash
kubectl get role #👈👈👈 Get all the roles defined in the namespace
```

```bash
kubectl describe role rbac-role #👈👈👈 Describe the role
```

```console
Name:         rbac-role
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources    Non-Resource URLs  Resource Names  Verbs
  ---------    -----------------  --------------  -----
  pods/status  []                 []              [get watch]
  pods         []                 []              [get watch]
```

```bash
kubectl edit role rbac-role #👈👈👈 Edit the role
```

```console
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  creationTimestamp: "2021-12-08T03:29:23Z"
  name: rbac-role
  namespace: rbac-namespace
  resourceVersion: "7927"
  uid: 08464e32-4994-4db5-804c-61dabaa803b1
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - pods/status
  verbs:
  - get
  - watch
  - delete #👈👈👈 Add the verb "delete"
```

```bash
kubectl describe role  rbac-role #👈👈👈 Describe the role
```

```console
Name:         rbac-role
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources    Non-Resource URLs  Resource Names  Verbs
  ---------    -----------------  --------------  -----
  pods/status  []                 []              [get watch delete]
  pods         []                 []              [get watch delete]
```

```bash
kubectl auth can-i get pods --as=system:serviceaccount:rbac-namespace:rbac-sa # yes to get
kubectl auth can-i delete pods --as=system:serviceaccount:rbac-namespace:rbac-sa # yes to delete
```

</p>
</details>
<br />

#### Clean Up

<details class="faq box"><summary>Clean Up</summary> 
<p>

```bash
yes | rm -R ~/ckad/
kubectl delete ns secret-namespace --force
kubectl delete ns quota-namespace --force
kubectl delete ns serviceaccount-namespace --force
kubectl delete ns rbac-namespace --force
```

</p>
</details>

_End of Section_
