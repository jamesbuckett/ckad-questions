## Sample CKAD Extension Questions and Answers

#### 05-01. Create a container from the attached Dockerfile and index.html. Name the image `my-image`. Name the container `my-container`. Run the image exposing port `8080` on the host and port `80` on the container. Stop the container. Delete the image.

<details><summary>show</summary>
<p>

Create a file called index.html
```bash
vi index.html

```

Edit index.html with the following text.
```bash
Hardships often prepare ordinary people for an extraordinary destiny.

```

Create a file called Dockerfile
```bash
vi Dockerfile

```

Edit the Docker with to include the text below
```bash
FROM nginx:latest
COPY ./index.html /usr/share/nginx/html/index.html
```

</p>
</details>

<details><summary>show</summary>
<p>

```bash
clear
# Build the docker image
docker build -t my-image:v0.1 .

```

```bash
clear
# Run the docker image
docker run -it --rm -d -p 8080:80 --name my-container my-image:v0.1

```

```bash
clear
# Verify Opertaion
curl localhost:8080

```

```bash
clear
# List all images
docker ps -a

```

```bash
clear
# Stop the Container
docker container stop my-container

```

```bash
clear
# Delete the Image
docker image rm my-image:v0.1

```

</p>
</details>

#### 05-02. Give the command to list out all the available API groups on your cluster. Then list out the API's in the `named` API (/apis) group.

<details><summary>show</summary>
<p>

```bash
clear
# Use the kubectl proxy to provide credentials to connect to the API server
# kubectl proxy starts a local proxy service on poort 8001
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
curl http://localhost:8001 -k | more 
```

Output:
```bash
1  "paths": [
00  593    "/.well-known/openid-configuration",
2     "/api",
      "/api/v1",
     "/apis",
0    "/apis/",
  593    "/apis/admissionregistration.k8s.io",
2     "/apis/admissionregistration.k8s.io/v1",
       "/apis/admissionregistration.k8s.io/v1beta1",
0         "/apis/apiextensions.k8s.io",
0  28    "/apis/apiextensions.k8s.io/v1",
96k         "/apis/apiextensions.k8s.io/v1beta1",
 0 -    "/apis/apiregistration.k8s.io",
-:--:-- -    "/apis/apiregistration.k8s.io/v1",
-:--:-- --:--:    "/apis/apiregistration.k8s.io/v1beta1",
-- 2    "/apis/apps",
89    "/apis/apps/v1",
6    "/apis/authentication.k8s.io",
k
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
# List all supported resource groups under the apis API
curl http://locahost:8001/apis -k | grep "name" | more

```

Output:
```bash
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
```


</p>
</details>

#### 05-03. List all the Custom Resource Definitions installed in a cluster.

<details><summary>show</summary>
<p>

```bash
clear
# kubectl get Custom Resource Defintions
kubectl get crds

```

Output
```bash
extensionservices.projectcontour.io           2021-09-22T06:28:39Z
httpproxies.projectcontour.io                 2021-09-22T06:28:39Z
tlscertificatedelegations.projectcontour.io   2021-09-22T06:28:39Z
```

</p>
</details>


*End of Section*