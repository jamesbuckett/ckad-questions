## Sample CKAD Services and Networking Q&A

### Services and Networking – 20%

- Demonstrate basic understanding of NetworkPolicies [\*\*](https://github.com/jamesbuckett/ckad-questions/blob/main/04-ckad-services-networking.md#04-01-create-a-namespace-called-netpol-namespace-create-a-pod-called-web-pod-using-the-nginx-image-and-exposing-port-80-label-the-pod-tierweb-create-a-pod-called-app-pod-using-the-nginx-image-and-exposing-port-80-label-the-pod-tierapp-create-a-pod-called-db-pod-using-the-nginx-image-and-exposing-port-80-label-the-pod-tierdb-create-a-network-policy-called-my-netpol-that-allows-the-web-pod-to-only-egress-to-app-pod-on-port-80-in-turn-only-allow-app-pod-to-egress-to-db-pod-on-port-80)
- Provide and troubleshoot access to applications via services [\*\*](https://github.com/jamesbuckett/ckad-questions/blob/main/04-ckad-services-networking.md#04-02-create-a-namespace-called-service-namespace-create-a-pod-called-service-pod-using-the-nginx-image-and-exposing-port-80-label-the-pod-tierweb-create-a-service-for-the-pod-called-my-service-allowing-for-communication-inside-the-cluster-let-the-service-expose-port-8080)
- Use Ingress rules to expose applications [\*\*](https://github.com/jamesbuckett/ckad-questions/blob/main/04-ckad-services-networking.md#04-03-create-an-ingress-called-my-ingress-to-expose-the-service-my-service-outside-the-cluster)

#### 04-01. Create a namespace called `netpol-namespace`. Create a pod called `web-pod` using the `nginx` image and exposing port `80`. Label the pod `tier=web`. Create a pod called `app-pod` using the `nginx` image and exposing port `80`. Label the pod `tier=app`. Create a pod called `db-pod` using the `nginx` image and exposing port `80`. Label the pod `tier=db`. Create a Network Policy called `my-netpol` that allows the `web-pod` to only egress to `app-pod` on port `80`.

```diff
Please NOTE:
- Docker Desktop does not support CNI (container network interface) so the NetworkPolicy's define are ignored.
- The commands work but the NetworkPolicy's are not enforced
```

I use the notepad to sketch out the ingress and egress before starting

Rules

- `tier: web` > `tier: app` on port 80

<details><summary>show</summary>
<p>

##### Prerequisites

```bash
mkdir ~/ckad/
clear
# Create all the required resources
kubectl create namespace netpol-namespace
kubectl config set-context --current --namespace=netpol-namespace

# tier: web
kubectl run web-pod --image=nginx --port=80  --labels="tier=web"
kubectl expose pod web-pod --port=80 --name=web-service

# tier: app
kubectl run app-pod --image=nginx --port=80 --labels="tier=app"
kubectl expose pod app-pod --port=80 --target-port=80 --name=app-service

# tier: db
kubectl run db-pod --image=nginx --port=80 --labels="tier=db"
kubectl expose pod db-pod --port=80 --target-port=80 --name=db-service

clear
kubectl get all
kubectl get pod -L tier
```

```bash
clear
# Test connectivity without  a Network Policy to app-service
kubectl exec web-pod -- curl -s app-service:80
```

```bash
clear
# Test connectivity without  a Network Policy to db-service
kubectl exec web-pod -- curl -s db-service:80
```

</p>
</details>

<details><summary>show</summary>
<p>

##### Solution

kubernetes.io bookmark: [The NetworkPolicy resource](https://kubernetes.io/docs/concepts/services-networking/network-policies/#networkpolicy-resource)

```bash
mkdir -p ~/ckad/
vi ~/ckad/04-01-netpol.yml
```

```bash
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: default-deny-all
spec:
  podSelector: {}
  ingress: []
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: web-netpol #👈👈👈 Change
spec:
  podSelector:
    matchLabels:
      tier: web #👈👈👈 Change - Which pod does this Network Policy Apply to i.e. any pod with label tier=web
  egress:
  - to:
    - podSelector:
        matchLabels:
          tier: app #👈👈👈 Egress - Traffic to pod with label tier=app
  policyTypes:
  - Egress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: app-netpol #👈👈👈 Change
spec:
  podSelector:
    matchLabels:
      tier: app #👈👈👈 Change - Which pod does this Network Policy Apply to i.e. any pod with label tier=app
  ingress:
  - from:
    - podSelector:
        matchLabels:
          tier: web #👈👈👈 Ingress - Traffic from pod with label tier=web
  policyTypes:
  - Ingress
```

```bash
kubectl apply -f ~/ckad/04-01-netpol.yml
```

```bash
clear
# Test connectivity with Network Policy
# Remember on Docker Desktop this will work as NetworkPolicy's are not enforced
kubectl exec web-pod -- curl -s app-service:80
```

```bash
clear
# Test connectivity with Network Policy
# Remember on Docker Desktop this will work as NetworkPolicy's are not enforced
kubectl exec web-pod -- curl -s db-service:80
```

</p>
</details>

#### 04-02. Create a namespace called `service-namespace`. Create a pod called `service-pod` using the `nginx` image and exposing port `80`. Label the pod `tier=web`. Create a service for the pod called `my-service` allowing for communication inside the cluster. Let the service expose port 8080.

<details><summary>show</summary>
<p>

##### Overview

![04-02](https://user-images.githubusercontent.com/18049790/136655759-a276fab5-fd7e-4703-91ca-56db086917ac.png)

</p>
</details>

<details><summary>show</summary>
<p>

##### Prerequisites

```bash
mkdir -p ~/ckad/
clear
kubectl create namespace service-namespace
kubectl config set-context --current --namespace=service-namespace
```

##### Help Examples

```bash
clear
kubectl run -h | more
```

Output:

```
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

<details><summary>show</summary>
<p>

##### Help Examples

```bash
clear
kubectl expose -h | more
```

Output:

```
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

#### 04-03. Create an ingress called `my-ingress` to expose the service `my-service` from previous question, outside the cluster.

<details><summary>show</summary>
<p>

##### Overview

![04-03-nginx](https://user-images.githubusercontent.com/18049790/136655897-148abcb7-4a6d-4d5b-afe0-18a377921d70.png)

![04-03-ing](https://user-images.githubusercontent.com/18049790/136655911-644a6aea-c237-4fee-aa07-8f75ac786b64.png)

</p>
</details>

<details><summary>show</summary>
<p>

##### Solution

kubernetes.io bookmark: [The Ingress resource](https://kubernetes.io/docs/concepts/services-networking/ingress/#the-ingress-resource)

```bash
mkdir -p ~/ckad/
vi ~/ckad/04-03-ing.yml
```

```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress #👈👈👈 Change: `my-ingress`
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
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

```
Name:             my-ingress
Namespace:        service-namespace
Address:          localhost
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

I sometimes had trouble with this networking setup. I just rebooted and this would work.

Output:

```
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

<details><summary>show</summary>
<p>

```bash
yes | rm -R ~/ckad/
kubectl delete ns service-namespace --force
kubectl delete ns netpol-namespace --force
```

</p>
</details>

_End of Section_
