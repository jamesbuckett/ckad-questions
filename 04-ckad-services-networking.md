## Sample CKAD Services and Networking Q&A

### Services and Networking – 20%

- Demonstrate basic understanding of NetworkPolicies [\*\*](https://github.com/jamesbuckett/ckad-questions/blob/main/04-ckad-services-networking.md#04-01-create-a-namespace-called-netpol-namespace-create-a-pod-called-web-pod-using-the-nginx-image-and-exposing-port-80-label-the-pod-tierweb-create-a-pod-called-app-pod-using-the-nginx-image-and-exposing-port-80-label-the-pod-tierapp-create-a-pod-called-db-pod-using-the-nginx-image-and-exposing-port-80-label-the-pod-tierdb-create-a-network-policy-called-my-netpol-that-allows-the-web-pod-to-only-egress-to-app-pod-on-port-80-in-turn-only-allow-app-pod-to-egress-to-db-pod-on-port-80)
- Provide and troubleshoot access to applications via services [\*\*](https://github.com/jamesbuckett/ckad-questions/blob/main/04-ckad-services-networking.md#04-02-create-a-namespace-called-service-namespace-create-a-pod-called-service-pod-using-the-nginx-image-and-exposing-port-80-label-the-pod-tierweb-create-a-service-for-the-pod-called-my-service-allowing-for-communication-inside-the-cluster-let-the-service-expose-port-8080)
- Use Ingress rules to expose applications [\*\*](https://github.com/jamesbuckett/ckad-questions/blob/main/04-ckad-services-networking.md#04-03-create-an-ingress-called-my-ingress-to-expose-the-service-my-service-outside-the-cluster)
<br />

#### 04-01. Network Policy Question
* Create a namespace called `netpol-namespace`. 
* Create a pod called `web-pod` using the `nginx` image and label the pod `tier=web`. 
* Create a pod called `app-pod` using the `nginx` image and label the pod `tier=app`. 
* Create a pod called `db-pod` using the `nginx` image and label the pod `tier=db`. 
* Create Network Policies that allow the `web-pod` to connect with the `app-pod` on port `80` only.

```diff
Please NOTE:
- Docker Desktop does not support CNI (container network interface) so the NetworkPolicy's define are ignored.
- The commands work but the NetworkPolicy's are not enforced
```

<details class="faq box"><summary>Overview</summary>
<p>

![05-netpol](https://user-images.githubusercontent.com/18049790/140638229-62871b17-bc71-4e51-a71c-4c75c178a78f.jpg)

I use the notepad to sketch out the ingress and egress before starting

Rules
- `tier: web` > `tier: app` on port 80

Use this link to visually solve the problem:
* [Network Policy Editor](https://editor.cilium.io/)

Use this link for common network policy recipes:
* [Kubernetes Network Policy Recipes](https://github.com/ahmetb/kubernetes-network-policy-recipes)

Notes 
* Network policies do not conflict; they are additive. 
* If any policy or policies select a pod, the pod is restricted to what is allowed by the union of those policies' ingress/egress rules. 
* Thus, order of evaluation does not affect the policy result.

</p>
</details>


<details class="faq box"><summary>Prerequisites</summary>
<p>

For clarity in the solution steps below i use images that return: 
* web-pod 
  * web-pod !!!
  * web-pod !!!
  * web-pod !!!
* app-pod
  * app-pod !!!
  * app-pod !!!
  * app-pod !!!
* db-pod
  * db-pod !!!
  * db-pod !!!
  * db-pod !!!

```bash
mkdir ~/ckad/
clear
# Create all the required resources
kubectl create namespace netpol-namespace
kubectl config set-context --current --namespace=netpol-namespace

# tier: web
kubectl run web-pod --image=docker.io/jamesbuckett/web:latest --port=80  --labels="tier=web"

# tier: app
kubectl run app-pod --image=docker.io/jamesbuckett/app:latest --port=80 --labels="tier=app"

# tier: db
kubectl run db-pod --image=docker.io/jamesbuckett/db:latest --port=80 --labels="tier=db"

clear
kubectl get all
kubectl get pod -L tier
```

```bash
clear
# Get the IP address of pods
kubectl get pods -o wide
```

```bash
# Inside the web-pod try to curl to app and db pods
kubectl exec --stdin --tty web-pod -- /bin/bash

#curl <app-pod-ip>
#curl <db-pod-ip>
```

</p>
</details>

<details class="faq box"><summary>Solution</summary>
<p>

kubernetes.io bookmark: [The NetworkPolicy resource](https://kubernetes.io/docs/concepts/services-networking/network-policies/#networkpolicy-resource)

Deny All Traffic in Namespace from [here](https://kubernetes.io/docs/concepts/services-networking/network-policies/#default-deny-all-ingress-and-all-egress-traffic)

```bash
cat << EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: netpol-namespace
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
EOF
  ```

```bash
mkdir -p ~/ckad/
vi ~/ckad/04-01-netpol.yml
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: web-policy  #👈👈👈 Change
  namespace: netpol-namespace
spec:
  podSelector:
    matchLabels:
      tier: web #👈👈👈 Change - Which pod does this Network Policy Apply to i.e. any pod with label tier=web
  egress:
    - to:
        - podSelector:
            matchLabels:
              tier: app #👈👈👈 Egress - Traffic to pod with label tier=app
      ports:
        - port: 80
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: app-policy #👈👈👈 Change  
  namespace: netpol-namespace
spec:
  podSelector:
    matchLabels:
      tier: app #👈👈👈 Change - Which pod does this Network Policy Apply to i.e. any pod with label tier=app
  ingress:
    - from:
        - podSelector:
            matchLabels:
              tier: web #👈👈👈 Ingress - Traffic from pod with label tier=web
      ports:
        - port: 80
```

```bash
kubectl apply -f ~/ckad/04-01-netpol.yml
```

```bash
clear
# Get the IP address of pods
kubectl get pods -o wide
```

```bash
# Inside the web-pod try to curl to app and db pods
kubectl exec --stdin --tty web-pod -- /bin/bash

#curl <app-pod-ip>
#curl <db-pod-ip>
```

Output: This output is from Digital Ocean where the Network Policies are enforced.

```console
[root@digital-ocean-droplet ~ (do-sgp1-digital-ocean-cluster:netpol-namespace)]# kubectl get pods -o wide
NAME      READY   STATUS    RESTARTS   AGE     IP             NODE                       NOMINATED NODE   READINESS GATES
app-pod   1/1     Running   0          57s     10.244.1.197   digital-ocean-pool-uch0o   <none>           <none>
db-pod    1/1     Running   0          2m13s   10.244.3.11    digital-ocean-pool-uch0j   <none>           <none>
web-pod   1/1     Running   0          2m13s   10.244.3.3     digital-ocean-pool-uch0j   <none>           <none>

[root@digital-ocean-droplet ~ (do-sgp1-digital-ocean-cluster:netpol-namespace)]# kubectl exec --stdin --tty web-pod -- /bin/bash
root@web-pod:/# curl 10.244.1.197
app-pod !!!
app-pod !!!
app-pod !!!
root@web-pod:/# curl 10.244.3.11
db-pod !!!
db-pod !!!
db-pod !!!
root@web-pod:/# curl 10.244.3.3
web-pod !!!
web-pod !!!
web-pod !!!
---------------------------------- #👈👈👈 Network Policy Applied
root@web-pod:/# curl 10.244.3.3
web-pod !!!
web-pod !!!
web-pod !!!
root@web-pod:/# curl 10.244.3.11 #👈👈👈 Cannot curl db-pod
^C
root@web-pod:/# curl 10.244.1.197 #👈👈👈 Can curl app-pod
app-pod !!!
app-pod !!!
app-pod !!!
```

</p>
</details>
<br />

#### 04-02. Kubernetes Service Question
* Create a namespace called `service-namespace`. 
* Create a pod called `service-pod` using the `nginx` image and exposing port `80`. 
* Label the pod `tier=web`. 
* Create a service for the pod called `my-service` allowing for communication inside the cluster. 
* Let the service expose port 8080.

<details class="faq box"><summary>Overview</summary>
<p>

![05-network-svc](https://user-images.githubusercontent.com/18049790/140637642-5ef46de4-4867-41a6-ba44-6333cd9441af.jpg)

</p>
</details>

<details class="faq box"><summary>Prerequisites</summary>
<p>

```bash
mkdir -p ~/ckad/
clear
kubectl create namespace service-namespace
kubectl config set-context --current --namespace=service-namespace
```

</p>
</details>

<details class="faq box"><summary>Solution - Create Pod</summary>
<p>

##### Help Examples

```bash
clear
kubectl run -h | more
```

Output:

```console
Examples:
  # Start a nginx pod
  kubectl run nginx --image=nginx

  # Start a hazelcast pod and let the container expose port 5701
  kubectl run hazelcast --image=hazelcast/hazelcast --port=5701 👈👈👈 This example matches most closely to the question: exposing port `80`

  # Start a hazelcast pod and set environment variables "DNS_DOMAIN=cluster" and "POD_NAMESPACE=default" in the container
  kubectl run hazelcast --image=hazelcast/hazelcast --env="DNS_DOMAIN=cluster" --env="POD_NAMESPACE=default"

  # Start a hazelcast pod and set labels "app=hazelcast" and "env=prod" in the container
  kubectl run hazelcast --image=hazelcast/hazelcast --labels="app=hazelcast,env=prod" 👈👈👈 This example matches most closely to the question: Label the pod `tier=web`

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

##### Solution

```bash
clear
kubectl run service-pod --image=nginx --port=80  --labels="tier=web"
kubectl get all
```

</p>
</details>

<details class="faq box"><summary>Solution - Expose Service</summary>
<p>

##### Help Examples

```bash
clear
kubectl expose -h | more
```

Output:

```console
Examples:
  # Create a service for a replicated nginx, which serves on port 80 and connects to the containers on port 8000
  kubectl expose rc nginx --port=80 --target-port=8000

  # Create a service for a replication controller identified by type and name specified in "nginx-controller.yaml",
which serves on port 80 and connects to the containers on port 8000
  kubectl expose -f nginx-controller.yaml --port=80 --target-port=8000

  # Create a service for a pod valid-pod, which serves on port 444 with the name "frontend"
  kubectl expose pod valid-pod --port=444 --name=frontend  👈👈👈 This example matches most closely to the question: pod called `my-service`

  # Create a second service based on the above service, exposing the container port 8443 as port 443 with the name
"nginx-https"
  kubectl expose service nginx --port=443 --target-port=8443 --name=nginx-https

  # Create a service for a replicated streaming application on port 4100 balancing UDP traffic and named 'video-stream'.
  kubectl expose rc streamer --port=4100 --protocol=UDP --name=video-stream

  # Create a service for a replicated nginx using replica set, which serves on port 80 and connects to the containers on
port 8000
  kubectl expose rs nginx --port=80 --target-port=8000 👈👈👈 This example matches most closely to the question: service expose port 8080

  # Create a service for an nginx deployment, which serves on port 80 and connects to the containers on port 8000
  kubectl expose deployment nginx --port=80 --target-port=8000
```

##### Solution

```bash
clear
kubectl expose pod service-pod --port=8080 --target-port=80 --name=my-service
clear
kubectl get pod -o wide
kubectl get service
kubectl get ep
```

</p>
</details>
<br />

#### 04-03. Ingress Question
* Create an ingress called `my-ingress` to expose the service `my-service` from previous question, outside the cluster.

<details class="faq box"><summary>Overview</summary>
<p>

![05-network-ing](https://user-images.githubusercontent.com/18049790/140637548-d1a9ced9-7c66-406c-86d3-1a7001de2e75.jpg)

</p>
</details>

<details class="faq box"><summary>Prerequisites</summary>
<p>

##### Prerequisites

Install the  Ingress Controller if you have not already installed it:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.3.0/deploy/static/provider/cloud/deploy.yaml
```

</p>
</details>

<details class="faq box"><summary>Solution</summary>
<p>

##### Solution

kubernetes.io bookmark: [The Ingress resource](https://kubernetes.io/docs/concepts/services-networking/ingress/#the-ingress-resource)

```bash
mkdir -p ~/ckad/
vi ~/ckad/04-03-ing.yml
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress #👈👈👈 Change: `my-ingress`
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: / #👈👈👈 Change
        pathType: Prefix
        backend:
          service:
            name: my-service #👈👈👈 Change: `my-service`
            port:
              number: 8080 #👈👈👈 Change: --port=8080          
```

```bash
clear
kubectl apply -f ~/ckad/04-03-ing.yml

# Describe the ingress
# This must be present to continue: Address: localhost
# If not shutdown Docker Desktop and reboot Windows 10
kubectl describe ingress my-ingress
```

Output:

```console
Name:             my-ingress
Namespace:        service-namespace
Address:          localhost #👈👈👈 This must be present for the solution to work
Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
Rules:
  Host        Path  Backends
  ----        ----  --------
  *
              /   my-service:8080 (10.1.1.37:80)
Annotations:  nginx.ingress.kubernetes.io/rewrite-target: /
Events:       <none>
```

```bash
clear
# Verify that the NGINX page is rendering via the Ingress endpoint
# If you have trouble with this reboot
curl localhost
```

Output:

```console
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
<br />

#### Clean Up

<details class="faq box"><summary>Clean Up</summary> 
<p>

```bash
yes | rm -R ~/ckad/
kubectl delete ns service-namespace --force
kubectl delete ns netpol-namespace --force
```

</p>
</details>

_End of Section_
