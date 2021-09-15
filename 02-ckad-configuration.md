## Sample CKAD Configuration Questions and Answers

#### 02-01. Create a secret called `my-secret` which contains `user=bob` and `password=123456`. Make the secret immutable. Make the secret available in a pod called `secret-pod` as environmental variables `SECRET-ENV-USER` and `SECRET-ENV-PASSWORD`. Create a pod called `secret-pod` using image `nginx` in namespace called `secret-namespace`. Create the namespace.

<details><summary>show</summary>
<p>

```bash
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
  kubectl create secret generic my-secret --from-literal=key1=supersecret --from-literal=key2=topsecret
  
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
```


```bash
# Apply the YAML file to the Kubernetes API server
# The secret is availiable to all pods in the namespace 
kubectl apply -f 02-01-secret.yml
# Verify that the secret got created
kubectl get secret my-secret
kubectl describe secret my-secret
```

```bash
# Now to create the pod that will consume the secret
kubectl run secret-pod --image=nginx --restart=Never -n secret-namespace --dry-run=client -o yaml > 02-01-pod.yml
vi 02-01.yml
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
```

```bash
# Apply the YAML file to the Kubernetes API server
kubectl apply -f 02-01.yml
```

```bash
# Quick verification that the deployment was created and is working
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

#### Clean Up 

```bash
kubectl delete ns secret-namespace
```

*End of Section*