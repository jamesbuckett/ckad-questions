## Sample CKAD Configuration Questions and Answers

#### 02-01. Create a namespace called `secret-namespace`. Create a secret in this namespace called `my-secret`. The secret should be immutable and contain the literal values `user=bob` and `password=123456`. Create a pod called called `secret-pod` using the `nginx` image. The pod should consume the secret as environmental variables `SECRET-ENV-USER` and `SECRET-ENV-PASSWORD`.

<details><summary>show</summary>
<p>

```bash
clear
kubectl create namespace secret-namespace
kubectl config set-context --current --namespace=secret-namespace
```

</p>
</details>

<details><summary>show</summary>
<p>

Three types of secret:
* generic
* docker-registry
* tls

```bash
clear
# kubectl create secret -h 
kubectl create secret generic -h | more
```

Output:
```bash
Examples:
  # Create a new secret named my-secret with keys for each file in folder bar
  kubectl create secret generic my-secret --from-file=path/to/bar
  
  # Create a new secret named my-secret with specified keys instead of names on disk
  kubectl create secret generic my-secret --from-file=ssh-privatekey=path/to/id_rsa
--from-file=ssh-publickey=path/to/id_rsa.pub
  
  # Create a new secret named my-secret with key1=supersecret and key2=topsecret
  kubectl create secret generic my-secret --from-literal=key1=supersecret --from-literal=key2=topsecret ### This example matches most closely to the question.
  
  # Create a new secret named my-secret using a combination of a file and a literal
  kubectl create secret generic my-secret --from-file=ssh-privatekey=path/to/id_rsa --from-literal=passphrase=topsecret
  
  # Create a new secret named my-secret from an env file
  kubectl create secret generic my-secret --from-env-file=path/to/bar.env
```

</p>
</details>

<details><summary>show</summary>
<p>

```bash
clear
# Create a generic secret
kubectl create secret generic my-secret --from-literal=user=bob --from-literal=password=123456 --dry-run=client -o yaml > 02-01-secret.yml
vi 02-01-secret.yml
```
kubernetes.io: [Immutable Secrets](https://kubernetes.io/docs/concepts/configuration/secret/#secret-immutable)

```bash
apiVersion: v1
data:
  password: MTIzNDU2
  user: Ym9i
immutable: true   # From Immutable Secrets link above
kind: Secret
metadata:
  creationTimestamp: null
  name: my-secret

# vi edits
# / - find
# d$ - delete to end of line
# :u - undo on any error
# :wq - write and quit  
```


```bash
clear
# Apply the YAML file to the Kubernetes API server
# The secret is availiable to all pods in the namespace 
kubectl apply -f 02-01-secret.yml
# Verify that the secret got created
kubectl get secret my-secret
kubectl describe secret my-secret
```

```bash
clear
# Now to create the pod that will consume the secret
kubectl run secret-pod --image=nginx --restart=Never -n secret-namespace --dry-run=client -o yaml > 02-01-pod.yml
vi 02-01-pod.yml
```

kubernetes.io: [Using Secrets as environment variables](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-environment-variables)

```bash
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
    env:                             # From Using Secrets as environment variables link above
      - name: SECRET-ENV-USER        # From Using Secrets as environment variables link above
        valueFrom:                   # From Using Secrets as environment variables link above
          secretKeyRef:              # From Using Secrets as environment variables link above
            name: my-secret          # From Using Secrets as environment variables link above
            key: user                # From Using Secrets as environment variables link above
      - name: SECRET-ENV-PASSWORD    # From Using Secrets as environment variables link above
        valueFrom:                   # From Using Secrets as environment variables link above
          secretKeyRef:              # From Using Secrets as environment variables link above
            name: my-secret          # From Using Secrets as environment variables link above
            key: password            # From Using Secrets as environment variables link above
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

# vi edits
# / - find
# d$ - delete to end of line
# :u - undo on any error
# :wq - write and quit
```

```bash
clear
# Apply the YAML file to the Kubernetes API server
kubectl apply -f 02-01-pod.yml
```

```bash
clear
# Quick verification that the deployment was created and the secret is visible as an environmetal variable
kubectl exec secret-pod -- env
```

Output:
```bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=secret-pod
NGINX_VERSION=1.21.3
NJS_VERSION=0.6.2
PKG_RELEASE=1~buster
SECRET-ENV-USER=bob           # Success
SECRET-ENV-PASSWORD=123456    # Success
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

#### 02-02. Create a namespace called `quota-namespace`. Create a Resource Quota for this namespace called `my-quota`. Set a memory reservation of `2Gi`. Set a CPU reservation of `500Mi`.

<details><summary>show</summary>
<p>

```bash
clear
kubectl create namespace quota-namespace
kubectl config set-context --current --namespace=quota-namespace
```

```bash
clear
kubectl create quota -h | more
```

Output
```bash
  # Create a new resource quota named my-quota
  kubectl create quota my-quota --hard=cpu=1,memory=1G,pods=2,services=3,replicationcontrollers=2,resourcequotas=1,secrets=5,persistentvolumeclaims=10 ### This example matches most closely to the question.
  
  # Create a new resource quota named best-effort
  kubectl create quota best-effort --hard=pods=100 --scopes=BestEffort
```

</p>
</details>

<details><summary>show</summary>
<p>

```bash
clear
kubectl create quota my-quota --hard=cpu=500Mi,memory=2G
kubectl get quota
```
Output:
```bash
NAME       AGE    REQUEST                      LIMIT
my-quota   118s   cpu: 0/500Mi, memory: 0/2G
```
In English
* REQUEST = Minimum (Request)
* LIMIT = Maximum (Limits)


```bash
clear
# Try to run a Pod with resource requests exceeding the quota
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml | kubectl set resources -f - --requests=cpu=1000m,memory=4Gi --limits=cpu=1000m,memory=4Gi --local -o yaml > 02-02.yml
kubectl apply -f 02-02.yml
# Error from server (Forbidden): error when creating "02-02.yml": pods "nginx" is forbidden: exceeded quota: my-quota, requested: memory=4Gi, used: memory=0, limited: memory=2G
```

```bash
clear
# Try to run a Pod with resource requests within the quota
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml | kubectl set resources -f - --requests=cpu=250m,memory=1Gi --limits=cpu=250m,memory=1Gi --local -o yaml > 02-02.yml
kubectl apply -f 02-02.yml
kubectl get all
kubectl get quota
```

Output:
```bash
NAME       AGE   REQUEST                           LIMIT
my-quota   19m   cpu: 250m/500Mi, memory: 1Gi/2G   
```

[Meaning of memory](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#meaning-of-memory) 
* Limits and requests for memory are measured in bytes.
* You can express memory as a plain integer or as a fixed-point number using one of these suffixes: E, P, T, G, M, k. 
* You can also use the power-of-two equivalents: Ei, Pi, Ti, Gi, Mi, Ki. 

In English:
* E/Ei = Exabyte
* P/Pi = Petabyte
* T/Ti = Terrabyte
* G/Gi = Gigabyte
* M/Mi = Megabyte
* k/Ki = Kilobyte

</p>
</details>

#### Clean Up 

<details><summary>show</summary>
<p>

```bash
kubectl delete ns secret-namespace --force
kubectl delete ns quota-namespace --force
```

</p>
</details>

*End of Section*