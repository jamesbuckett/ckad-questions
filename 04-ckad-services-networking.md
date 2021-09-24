## Sample CKAD Services and Networking - 20% - Questions and Answers

### Services and Networking – 20%

- Demonstrate basic understanding of NetworkPolicies [\*\*](https://github.com/jamesbuckett/ckad-questions/blob/main/03-ckad-deployment.md#03-01-create-a-namespace-called-deployment-namespace-create-a-deployment-called-my-deployment-with-three-replicas-using-the-nginx-image-inside-the-namespace-expose-port-80-for-the-nginx-container-the-containers-should-be-named-my-container-each-container-should-have-a-memory-request-of-25mi-and-a-memory-limit-of-100mi)
- Provide and troubleshoot access to applications via services
- Use Ingress rules to expose applications [\*\*](https://github.com/jamesbuckett/ckad-questions/blob/main/04-ckad-services-networking.md#04-01-create-a-namespace-called-service-namespace-create-a-pod-called-service-pod-using-the-nginx-image-and-exposing-port-80-label-the-pod-tierweb-create-a-service-for-the-pod-called-my-service-allowing-for-communication-inside-the-cluster-let-the-service-expose-port-8080-create-an-ingress-called-my-ingress-to-expose-the-service-outside-the-cluster)

#### 04-01. Create a namespace called `service-namespace`. Create a pod called `service-pod` using the `nginx` image and exposing port `80`. Label the pod `tier=web`. Create a service for the pod called `my-service` allowing for communication inside the cluster. Let the service expose port 8080. Create an ingress called `my-ingress` to expose the service outside the cluster.

<details><summary>show</summary>
<p>

```bash
clear
kubectl create namespace service-namespace
kubectl config set-context --current --namespace=service-namespace
```

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
clear
kubectl run service-pod --image=nginx --port=80  --labels="tier=web"
kubectl get all
```

</p>
</details>

<details><summary>show</summary>
<p>

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
clear
kubectl expose pod service-pod --port=8080 --target-port=80 --name=my-service
kubectl get all
kubectl get ep
```

</p>
</details>

<details><summary>show</summary>
<p>

kubernetes.io [The Ingress resource](https://kubernetes.io/docs/concepts/services-networking/ingress/#the-ingress-resource)

```bash
vi q04-01-ing.yml
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
kubectl apply -f q04-01-ing.yml

# Pod Address under IP heading
kubectl get pod -o wide

# The Pod is an endpoint listed under ENDPOINTS with port :80
kubectl get ep

# The Service IP listed under CLUSTER-IP with PORT(S) :8080
kubectl get service -o wide

# Ingress IP listed under ADDRESS is localhost`
kubectl get ingress
```

```bash
# Verify that the NGINX page is rendering via the Ingress endpoint
curl localhost
```

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

#### 04-02. UNDER CONSTRUCTION. Create a namespace called `netpol-namespace`. Create a pod called `web-pod` using the `nginx` image and exposing port `80`. Label the pod `tier=web`. Create a pod called `db-pod-1` using the `nginx` image and exposing port `11`. Label the pod `tier=db-1`. Create a pod called `db-pod-2` using the `nginx` image and exposing port `22`. Label the pod `tier=db-2`. Create a Network Policy called `my-netpol` that allows the `web-pod` to only connect to `db-pod-1` on port `11` and to connect to `db-pod-2` on port `22`.

I use the notepad to sketch out the ingress and egress before starting

- `web-pod` > `db-pod-1` on port 11
- `web-pod` > `db-pod-2` on port 22

<details><summary>show</summary>
<p>

```bash
clear
# Create all the required resources
kubectl create namespace netpol-namespace
kubectl config set-context --current --namespace=netpol-namespace
kubectl run web-pod --image=nginx --port=80  --labels="tier=web"
kubectl expose pod web-pod --port=8080 --name=web-service
kubectl run db-pod-1 --image=docker.io/jamesbuckett/db-pod-1:latest --port=11 --labels="tier=db-1"
kubectl expose pod db-pod-1 --port=1111 --target-port=11 --name=db-pod-1-service
kubectl run db-pod-2 --image=docker.io/jamesbuckett/db-pod-2:latest --port=22 --labels="tier=db-2"
kubectl expose pod db-pod-2 --port=2222 --target-port=22 --name=db-pod-2-service
kubectl run db-pod-3 --image=nginx --port=33 --labels="tier=db-3"
kubectl expose pod db-pod-3 --port=3333 --target-port=33 --name=db-pod-3-service
clear
kubectl get all
kubectl get pod -L tier
```

```bash
clear
# Test connectivity without Network Policy
kubectl get pod -o wide
# kubectl exec web-pod -- curl -s <IP>:<PORT>
```

</p>
</details>

<details><summary>show</summary>
<p>

kubernetes.io: [The NetworkPolicy resource](https://kubernetes.io/docs/concepts/services-networking/network-policies/#networkpolicy-resource)

```bash
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5978
```

</p>
</details>

<details><summary>show</summary>
<p>

kubernetes.io: [The NetworkPolicy resource](https://kubernetes.io/docs/concepts/services-networking/network-policies/#networkpolicy-resource)

```bash
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: my-netpol     # Change
spec:
  podSelector:
    matchLabels:
      tier: web       # Change - Which pod does this Netowork Policy Apply to i.e. any pod with label tier=web
  policyTypes:
  - Egress
  egress:             # Egress - Traffic outwards from pod with label tier=web
  - to:               # First condition "to" podSelector and ports
    - podSelector:      # Condition podSelector
        matchLabels:
          tier: db-1      # First podSelector possibility
    - podSelector:
        matchLabels:
          tier: db-2      # Second podSelector possibility
    ports:              # Condition ports
    - protocol: TCP
      port: 11            # First ports possibility
    - protocol: TCP
      port: 22            # Second ports possibility

```

```bash
clear
# Test connectivity with Network Policy
kubectl apply -f
kubectl get pod -o wide | awk 'FNR == 2 {print $6}' | xargs -d'\n' curl
kubectl get pod -o wide | awk 'FNR == 3 {print $6}' | xargs -d'\n' curl
kubectl get pod -o wide | awk 'FNR == 4 {print $6}' | xargs -d'\n' curl
kubectl get pod -o wide | awk 'FNR == 5 {print $6}' | xargs -d'\n' curl
```

Read this as:

- Allow outgoing traffic if:
  - Destination Pod has label db-1 OR db-2  
    AND
  - Destination Port is 11 OR Destination Port is 22

Pod web=tier can connect to pod db-2 on port 11

</p>
</details>

#### Clean Up

<details><summary>show</summary>
<p>

```bash
kubectl delete ns service-namespace --force
kubectl delete ns netpol-namespace --force
```

</p>
</details>

_End of Section_
