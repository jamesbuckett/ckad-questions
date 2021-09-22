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
# Use the kubectl to provide credentials to connect to the API server
kubectl proxy &

```

```bash
clear
# List all available API groups
curl http://locahost:8001 -k | more

```

Output:
```bash



```

</p>
</details>

<details><summary>show</summary>
<p>

```bash
clear
# List all supported resource groups
curl http://locahost:8001/apis -k | grep "name"

```

Output:
```bash



```


</p>
</details>

*End of Section*