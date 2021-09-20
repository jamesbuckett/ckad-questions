## Sample CKAD Extension Questions and Answers

#### 05-01. Create a container from the attached Dockerfile and index.html. Name the image `my-image`. Name the container `my-container`. Run the container exposing port `8080` on the host and port `80` on the container. Stop the container. Delete the container.

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

<details><summary>show</summary>
<p>

```bash
# Build the docker image
docker build -t my-image:v0.1 .

```

```bash
# Run the docker image
docker run -it --rm -d -p 8080:80 --name my-container my-image

```

```bash
# Verify Opertaion
curl localhost:8080

```

```bash
# List all images
docker ps -a

```

```bash
# Stop the Container
docker container stop my-container

```

```bash
# Delete the Image
docker image rm my-image

```

</p>
</details>

*End of Section*