## Sample CKAD Application Deployment Q&A

### Application Deployment – 20%

- Understand Deployments and how to perform rolling updates [\*\*](https://github.com/jamesbuckett/ckad-questions/blob/main/03-ckad-deployment.md#03-01-create-a-namespace-called-deployment-namespace-create-a-deployment-called-my-deployment-with-three-replicas-using-the-nginx-image-inside-the-namespace-expose-port-80-for-the-nginx-container-the-containers-should-be-named-my-container-each-container-should-have-a-memory-request-of-25mi-and-a-memory-limit-of-100mi)
- Use the Helm package manager to deploy existing packages [\*\*](https://github.com/jamesbuckett/ckad-questions/blob/main/03-ckad-deployment.md#03-04-use-helm-to-install-wordpress-into-a-namespace-called-wordpress-namespace)
- Use Kubernetes primitives to implement common deployment strategies (e.g. blue/green or canary) [\*\*](https://github.com/jamesbuckett/ckad-questions/blob/main/03-ckad-deployment.md#03-05-create-a-namespace-called-blue-green-namespace-create-a-deployment-called-blue-deployment-with-10-replicas-using-the-nginx-image-inside-the-namespace-expose-port-80-for-the-nginx-containers-label-the-pods-versionblue-and-tierweb-create-a-service-called-bsg-service-to-route-traffic-to-blue-deployment-verify-that-traffic-is-flowing-from-the-service-to-the-deployment-create-a-new-deployment-called-green-deployment--with-10-replicas-using-the-nginx-image-inside-the-namespace-expose-port-80-for-the-nginx-containers-label-the-pods-versiongreen-and-tierweb-once-the-green-deployment-is-active-split-traffic-between-blue-deployment70-and-green-deployment30)

#### 03-01. Create a namespace called `deployment-namespace`. Create a Deployment called `my-deployment`, with `three` replicas, using the `nginx` image inside the namespace. Expose `port 80` for the nginx container. The containers should be named `my-container`. Each container should have a `memory request` of 25Mi and a `memory limit` of 100Mi.

<details><summary>show</summary>
<p>

##### Overview

![03-01-deploy](https://user-images.githubusercontent.com/18049790/136654817-cf8b912c-7dc3-486d-8615-6b8ecbdc0e8a.png)

![03-01-pod](https://user-images.githubusercontent.com/18049790/136654820-9ff806cf-5987-4eab-bfee-aaaf36e3837f.png)

</p>
</details>

<details><summary>show</summary>
<p>

##### Prerequisites

```bash
mkdir ~/ckad/
clear
# Create the namespace
kubectl create namespace deployment-namespace
```

```bash
clear
# Switch context into the namespace so that all subsequent commands execute inside that namespace.
kubectl config set-context --current --namespace=deployment-namespace
```

##### Help Examples

```bash
clear
# Run the help flag to get examples
# kubectl create deployment -h
kubectl create deploy -h | more
```

Output:

```
Examples:
  # Create a deployment named my-dep that runs the busybox image
  kubectl create deployment my-dep --image=busybox

  # Create a deployment with a command
  kubectl create deployment my-dep --image=busybox -- date

  # Create a deployment named my-dep that runs the nginx image with 3 replicas
  kubectl create deployment my-dep --image=nginx --replicas=3 👈👈👈 This example matches most closely to the question: `three` replicas

  # Create a deployment named my-dep that runs the busybox image and expose port 5701
  kubectl create deployment my-dep --image=busybox --port=5701 👈👈👈 This example matches most closely to the question: `port 80`
```

</p>
</details>

<details><summary>show</summary>
<p>

##### Solution

```bash
clear
# Using the best example that matches the question
kubectl create deployment my-deployment --image=nginx --replicas=3 --port=80 --dry-run=client -o yaml > ~/ckad/03-01.yml
```

```bash
# Edit the YAML file to make required changes
vi ~/ckad/03-01.yml
```

kubernetes.io: [Meaning of memory](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#meaning-of-memory)

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: my-deployment
  name: my-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-deployment
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: my-deployment
    spec:
      containers:
      - image: nginx
        ports:
        - containerPort: 80
        name: my-container  #👈👈👈 Change from nginx to my container
        resources:          #👈👈👈 From Meaning of memory link above
          requests:         #👈👈👈 From Meaning of memory link above
            memory: "25Mi"  #👈👈👈 From Meaning of memory link above
          limits:           #👈👈👈 From Meaning of memory link above
            memory: "100Mi" #👈👈👈 From Meaning of memory link above
status: {}
```

```bash
clear
# Apply the YAML file to the Kubernetes API server
kubectl apply -f ~/ckad/03-01.yml
```

```bash
clear
# Quick verification that the deployment was created and is working
kubectl get all
```

Output:

```
NAME                               READY   STATUS    RESTARTS   AGE
pod/my-deployment-67fc8546-9b4bm   1/1     Running   0          16m
pod/my-deployment-67fc8546-mjw24   1/1     Running   0          16m
pod/my-deployment-67fc8546-tp5bk   1/1     Running   0          16m

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-deployment   3/3     3            3           16m

NAME                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/my-deployment-67fc8546   3         3         3       16m
```

 </p>
</details>

#### 03-02. In the previous question a Deployment called `my-deployment` was created. Allow network traffic to flow to this deployment from inside the cluster on `port 8080`.

<details><summary>show</summary>
<p>

##### Overview

![03-02](https://user-images.githubusercontent.com/18049790/136654933-3cbb67a7-c892-402b-a7a1-9e6f2977ab51.png)

</p>
</details>

<details><summary>show</summary>
<p>

##### Help Examples

```bash
clear
# Run the help flag to get examples
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
  kubectl expose pod valid-pod --port=444 --name=frontend

  # Create a second service based on the above service, exposing the container port 8443 as port 443 with the name
"nginx-https"
  kubectl expose service nginx --port=443 --target-port=8443 --name=nginx-https

  # Create a service for a replicated streaming application on port 4100 balancing UDP traffic and named 'video-stream'.
  kubectl expose rc streamer --port=4100 --protocol=UDP --name=video-stream

  # Create a service for a replicated nginx using replica set, which serves on port 80 and connects to the containers on
port 8000
  kubectl expose rs nginx --port=80 --target-port=8000

  # Create a service for an nginx deployment, which serves on port 80 and connects to the containers on port 8000
  kubectl expose deployment nginx --port=80 --target-port=8000 👈👈👈 This example matches most closely to the question.
```

</p>
</details>

<details><summary>show</summary>
<p>

##### Solution

```bash
clear
# Using the best example that matches the question
kubectl expose deployment my-deployment --port=8080 --target-port=80
```

Watch out for the statement from inside the Cluster so this is of type: ClusterIP

Types include:

- ClusterIP (default)
- NodePort
- LoadBalancer
- ExternalName

```bash
clear
# Check that the Service was created
  # Inside the namespace: my-deployment
  # Outside the namespace: my-deployment.deployment-namespace.svc.cluster.local
kubectl get service
```

Output:

```
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
my-deployment   ClusterIP   10.245.79.74   <none>        80/TCP    103s
```

```bash
clear
# A quicker check is to see if the Pod Endpoints are being load balanced
kubectl get endpoints
kubectl get pods -o wide
```

Output:

```
NAME            ENDPOINTS                                         AGE
my-deployment   10.244.0.250:80,10.244.1.132:80,10.244.1.246:80   5m20s
# The three replicas internal endpoints are registered
```

</p>
</details>

#### 03-03. Create a namespace called `edit-namespace`. Create a deployment called `edit-deployment` with `2` replicas using the `redis` image in namespace. After the deployment is running, alter the containers to use the `nginx` image. Then alter the containers to use the `nginx:1.14.2` image and record the change.

<details><summary>show</summary>
<p>

##### Overview

![03-03](https://user-images.githubusercontent.com/18049790/136655071-9e4a9225-3a8f-4aed-8556-0767162fec36.png)

</p>
</details>

<details><summary>show</summary>
<p>

##### Prerequisites

```bash
clear
kubectl create namespace edit-namespace
kubectl create deployment edit-deployment --image=redis --replicas=2 -n edit-namespace
kubectl config set-context --current --namespace=edit-namespace
```

</p>
</details>

<details><summary>show</summary>
<p>

##### Solution

```bash
kubectl edit deployment.apps/edit-deployment
```

```bash
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2021-09-24T06:23:27Z"
  generation: 1
  labels:
    app: edit-deployment
  name: edit-deployment
  namespace: edit-namespace
  resourceVersion: "7856"
  uid: d482067c-da5f-43ce-aa31-25defd2d0de3
spec:
  progressDeadlineSeconds: 600
  replicas: 2
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: edit-deployment
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: edit-deployment
    spec:
      containers:
      - image: redis #👈👈👈 Change this to nginx
        imagePullPolicy: Always
        name: redis #👈👈👈 This is the catch, when you created the deployment it used the image=redis to also name the container redis
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
```

```bash
clear
# Check the image in the Deployment
kubectl describe deployment edit-deployment | grep Image
```

This works but does not record what the change was.

</p>
</details>

<details><summary>show</summary>
<p>

##### Solution

kubernetes.io:[Updating a Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment)

```bash
clear
# Use the kubectl set image command
kubectl set image deployment.apps/edit-deployment redis=nginx:1.14.2 --record
```

```bash
clear
# Check the image in the Deployment
kubectl describe deployment edit-deployment | grep Image
# Check that the change was recorded
kubectl rollout history deployment.apps/edit-deployment
```

</p>
</details>

#### 03-04. Use Helm to install WordPress into a namespace called `wordpress-namespace`

<details><summary>show</summary>
<p>

##### Overview

![03-04-wp](https://user-images.githubusercontent.com/18049790/136655517-2bf660fb-8f97-4503-b6b8-cae0bc4fc4b2.png)

![03-04-deploy](https://user-images.githubusercontent.com/18049790/136655521-cb6bba5d-e1f6-4e9f-876c-b8da98d96c64.png)

</p>
</details>

<details><summary>show</summary>
<p>

WordPress on Bitnami [details](https://github.com/bitnami/charts/tree/master/bitnami/wordpress/#installing-the-chart)

##### Prerequisites

```bash
clear
kubectl create namespace wordpress-namespace
kubectl config set-context --current --namespace=wordpress-namespace
```

```bash
cat << EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: wp-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-wp-release-wordpress
            port:
              number: 80
EOF
```

</p>
</details>

<details><summary>show</summary>
<p>

##### Solution

```bash
clear
# Add the Bitnami repo
helm repo add bitnami https://charts.bitnami.com/bitnami
```

```bash
# Search the Bitnami repo for available software
helm search repo bitnami
```

Output:

```bash
NAME                                            CHART VERSION   APP VERSION     DESCRIPTION
bitnami/bitnami-common                          0.0.9           0.0.9           DEPRECATED Chart with custom templates used in ...
bitnami/airflow                                 11.0.8          2.1.4           Apache Airflow is a platform to programmaticall...
bitnami/apache                                  8.8.3           2.4.50          Chart for Apache HTTP Server
bitnami/argo-cd                                 2.0.4           2.1.3           Declarative, GitOps continuous delivery tool fo...
bitnami/argo-workflows                          0.1.1           3.1.13          Argo Workflows is meant to orchestrate Kubernet...
bitnami/aspnet-core                             1.3.18          3.1.19          ASP.NET Core is an open-source framework create...
bitnami/cassandra                               8.0.6           4.0.1           Apache Cassandra is a free and open-source dist...
bitnami/cert-manager                            0.1.21          1.5.4           Cert Manager is a Kubernetes add-on to automate...
bitnami/common                                  1.10.0          1.10.0          A Library Helm Chart for grouping common logic ...
bitnami/concourse                               0.1.7           7.5.0           Concourse is a pipeline-based continuous thing-...
bitnami/consul                                  9.3.8           1.10.3          Highly available and distributed service discov...
bitnami/contour                                 5.7.0           1.18.2          Contour Ingress controller for Kubernetes
bitnami/contour-operator                        0.1.1           1.18.2          The Contour Operator extends the Kubernetes API...
bitnami/dataplatform-bp1                        8.0.1           0.0.11          OCTO Data platform Kafka-Spark-Solr Helm Chart
bitnami/dataplatform-bp2                        8.0.3           0.0.10          OCTO Data platform Kafka-Spark-Elasticsearch He...
bitnami/discourse                               5.0.2           2.7.8           A Helm chart for deploying Discourse to Kubernetes
bitnami/dokuwiki                                11.2.8          20200729.0.0    DokuWiki is a standards-compliant, simple to us...
bitnami/drupal                                  10.3.4          9.2.7           One of the most versatile open source content m...
bitnami/ejbca                                   3.0.1           7.4.3-2         Enterprise class PKI Certificate Authority buil...
bitnami/elasticsearch                           17.1.0          7.14.2          A highly scalable open-source full-text search ...
bitnami/etcd                                    6.8.4           3.5.0           etcd is a distributed key value store that prov...
bitnami/external-dns                            5.4.10          0.10.0          ExternalDNS is a Kubernetes addon that configur...
bitnami/fluentd                                 4.2.3           1.14.1          Fluentd is an open source data collector for un...
bitnami/geode                                   0.1.0           1.14.0          Apache Geode is a data management platform that...
bitnami/ghost                                   14.0.22         4.17.1          A simple, powerful publishing platform that all...
bitnami/grafana                                 6.3.2           8.2.0           Grafana is an open source, feature rich metrics...
bitnami/grafana-operator                        1.1.4           3.10.3          Kubernetes Operator based on the Operator SDK f...
bitnami/grafana-tempo                           0.2.7           1.1.0           Grafana Tempo is an open source, easy-to-use an...
bitnami/haproxy                                 0.2.13          2.4.7           HAProxy is a TCP proxy and a HTTP reverse proxy...
bitnami/harbor                                  11.0.4          2.3.3           Harbor is an an open source trusted cloud nativ...
bitnami/influxdb                                2.3.15          2.0.9           InfluxDB&trade; is an open source time-series d...
bitnami/jasperreports                           11.0.6          7.8.0           The JasperReports server can be used as a stand...
bitnami/jenkins                                 8.0.14          2.303.1         The leading open source automation server
bitnami/joomla                                  10.1.24         3.10.2          PHP content management system (CMS) for publish...
bitnami/jupyterhub                              0.1.20          1.4.2           JupyterHub brings the power of notebooks to gro...
bitnami/kafka                                   14.2.1          2.8.1           Apache Kafka is a distributed streaming platform.
bitnami/keycloak                                5.1.2           15.0.2          Keycloak is a high performance Java-based ident...
bitnami/kiam                                    0.3.15          3.6.0           kiam is a proxy that captures AWS Metadata API ...
bitnami/kibana                                  9.0.6           7.14.2          Kibana is an open source, browser based analyti...
bitnami/kong                                    4.1.4           2.6.0           Kong is a scalable, open source API layer (aka ...
bitnami/kube-prometheus                         6.1.12          0.51.2          kube-prometheus collects Kubernetes manifests t...
bitnami/kube-state-metrics                      2.1.11          2.2.1           kube-state-metrics is a simple service that lis...
bitnami/kubeapps                                7.5.7           2.4.1           Kubeapps is a dashboard for your Kubernetes clu...
bitnami/kubernetes-event-exporter               1.1.15          0.10.0          This tool allows exporting the often missed Kub...
bitnami/kubewatch                               3.2.16          0.1.0           Kubewatch is a Kubernetes watcher that currentl...
bitnami/logstash                                3.6.10          7.15.0          Logstash is an open source, server-side data pr...
bitnami/magento                                 19.0.4          2.4.3           A feature-rich flexible e-commerce solution. It...
bitnami/mariadb                                 9.6.2           10.5.12         Fast, reliable, scalable, and easy to use open-...
bitnami/mariadb-cluster                         1.0.2           10.2.14         DEPRECATED Chart to create a Highly available M...
bitnami/mariadb-galera                          6.0.1           10.6.4          MariaDB Galera is a multi-master database clust...
bitnami/mean                                    6.1.2           4.6.2           DEPRECATED MEAN is a free and open-source JavaS...
bitnami/mediawiki                               12.3.15         1.36.2          Extremely powerful, scalable software and a fea...
bitnami/memcached                               5.15.5          1.6.12          Chart for Memcached
bitnami/metallb                                 2.5.6           0.10.3          The Metal LB for Kubernetes
bitnami/metrics-server                          5.10.4          0.5.1           Metrics Server is a cluster-wide aggregator of ...
bitnami/minio                                   8.1.9           2021.10.6       Bitnami Object Storage based on MinIO&reg; is a...
bitnami/mongodb                                 10.27.2         4.4.9           NoSQL document-oriented database that stores JS...
bitnami/mongodb-sharded                         3.9.8           4.4.9           NoSQL document-oriented database that stores JS...
bitnami/moodle                                  11.1.2          3.11.3          Moodle&trade; is a learning platform designed t...
bitnami/mxnet                                   2.3.16          1.8.0           A flexible and efficient library for deep learning
bitnami/mysql                                   8.8.8           8.0.26          Chart to create a Highly available MySQL cluster
bitnami/nats                                    6.4.10          2.6.1           An open-source, cloud-native messaging system
bitnami/nginx                                   9.5.7           1.21.3          Chart for the nginx server
bitnami/nginx-ingress-controller                7.6.21          0.48.1          Chart for the nginx Ingress controller
bitnami/node                                    15.2.28         14.18.0         Event-driven I/O server-side JavaScript environ...
bitnami/node-exporter                           2.3.9           1.2.2           Prometheus exporter for hardware and OS metrics...
bitnami/oauth2-proxy                            1.0.2           7.1.3           A reverse proxy and static file server that pro...
bitnami/odoo                                    19.0.9          14.0.20210910   A suite of web based open source business apps.
bitnami/opencart                                10.0.26         3.0.3-8         A free and open source e-commerce platform for ...
bitnami/orangehrm                               10.1.23         4.8.0-0         OrangeHRM is a free HR management system that o...
bitnami/osclass                                 11.0.16         4.4.0           Osclass is a php script that allows you to quic...
bitnami/owncloud                                10.2.27         10.8.0          A file sharing server that puts the control and...
bitnami/parse                                   15.0.10         4.10.4          Parse is a platform that enables users to add a...
bitnami/phabricator                             11.0.30         2021.26.0       DEPRECATED Collection of open source web applic...
bitnami/phpbb                                   10.1.24         3.3.5           Community forum that supports the notion of use...
bitnami/phpmyadmin                              8.2.16          5.1.1           phpMyAdmin is an mysql administration frontend
bitnami/postgresql                              10.12.2         11.13.0         Chart for PostgreSQL, an object-relational data...
bitnami/postgresql-ha                           7.10.1          11.13.0         Chart for PostgreSQL with HA architecture (usin...
bitnami/prestashop                              13.2.3          1.7.8-0         A popular open source ecommerce solution. Profe...
bitnami/prometheus-operator                     0.31.1          0.41.0          DEPRECATED The Prometheus Operator for Kubernet...
bitnami/pytorch                                 2.3.14          1.9.0           Deep learning platform that accelerates the tra...
bitnami/rabbitmq                                8.22.4          3.9.7           Open source message broker software that implem...
bitnami/rabbitmq-cluster-operator               0.1.6           1.9.0           The RabbitMQ Cluster Kubernetes Operator automa...
bitnami/redis                                   15.4.1          6.2.6           Open source, advanced key-value store. It is of...
bitnami/redis-cluster                           6.3.9           6.2.6           Open source, advanced key-value store. It is of...
bitnami/redmine                                 17.0.9          4.2.2           A flexible project management web application.
bitnami/solr                                    2.0.7           8.9.0           Apache Solr is an open source enterprise search...
bitnami/spark                                   5.7.4           3.1.2           Spark is a fast and general-purpose cluster com...
bitnami/spring-cloud-dataflow                   4.1.1           2.8.2           Spring Cloud Data Flow is a microservices-based...
bitnami/sugarcrm                                1.0.6           6.5.26          DEPRECATED SugarCRM enables businesses to creat...
bitnami/suitecrm                                9.3.25          7.11.22         SuiteCRM is a completely open source enterprise...
bitnami/tensorflow-inception                    3.3.2           1.13.0          DEPRECATED Open-source software library for ser...
bitnami/tensorflow-resnet                       3.2.15          2.6.0           Open-source software library serving the ResNet...
bitnami/testlink                                9.2.24          1.9.20          Web-based test management system that facilitat...
bitnami/thanos                                  6.0.12          0.23.1          Thanos is a highly available metrics system tha...
bitnami/tomcat                                  9.4.3           10.0.12         Chart for Apache Tomcat
bitnami/wavefront                               3.1.12          1.7.1           Chart for Wavefront Collector for Kubernetes
bitnami/wavefront-adapter-for-istio             1.0.8           0.1.5           Wavefront Adapter for Istio is a lightweight Is...
bitnami/wavefront-hpa-adapter                   0.1.5           0.9.8           Wavefront HPA Adapter for Kubernetes is a Kuber...
bitnami/wavefront-prometheus-storage-adapter    1.0.8           1.0.3           Wavefront Storage Adapter is a Prometheus integ...
bitnami/wildfly                                 11.1.3          24.0.1          Chart for Wildfly
bitnami/wordpress                               12.1.20         5.8.1           Web publishing platform for building blogs and ... #👈👈👈
bitnami/zookeeper                               7.4.6           3.7.0           A centralized service for maintaining configura...
```

```bash
# Search the Bitnami repo for WordPress
helm search repo bitnami | grep wordpress
```

Output:

```bash
bitnami/wordpress                               12.1.20         5.8.1           Web publishing platform for building blogs and ...
```

```bash
# Install WordPress with Helm
helm install my-wp-release \
  --set wordpressUsername=admin \
  --set wordpressPassword=password \
  --set mariadb.auth.rootPassword=secretpassword \
  --set service.type=ClusterIP \
    bitnami/wordpress
```

Output:

```bash
NAME: my-wp-release
LAST DEPLOYED: Fri Oct  8 15:44:30 2021
NAMESPACE: wordpress-namespace
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
** Please be patient while the chart is being deployed **

Your WordPress site can be accessed through the following DNS name from within your cluster:

    my-wp-release-wordpress.wordpress-namespace.svc.cluster.local (port 80)

To access your WordPress site from outside the cluster follow the steps below:

1. Get the WordPress URL by running these commands:

   kubectl port-forward --namespace wordpress-namespace svc/my-wp-release-wordpress 80:80 &
   echo "WordPress URL: http://127.0.0.1//"
   echo "WordPress Admin URL: http://127.0.0.1//admin"

2. Open a browser and access WordPress using the obtained URL.

3. Login with the following credentials below to see your blog:

  echo Username: admin
  echo Password: $(kubectl get secret --namespace wordpress-namespace my-wp-release-wordpress -o jsonpath="{.data.wordpress-password}" | base64 --decode)
```

```bash
# Verify operation of WordPress
curl localhost:80
```

```bash
# List active helm releases
helm ls
```

```bash
# Delete WordPress with Helm
helm delete my-wp-release --purge
```

</p>
</details>

#### 03-05. Create a namespace called `blue-green-namespace`. Create a Deployment called `blue-deployment`, with `10` replicas, using the `nginx` image inside the namespace. Expose `port 80` for the nginx containers. Label the pods `version=blue` and `tier=web`. Create a Service called `bsg-service` to route traffic to `blue-deployment`. Verify that traffic is flowing from the Service to the Deployment. Create a new Deployment called `green-deployment` , with `10` replicas, using the `nginx` image inside the namespace. Expose `port 80` for the nginx containers. Label the pods `version=green` and `tier=web`. Once the `green-deployment` is active split traffic between `blue-deployment`=70% and `green-deployment`=30%

<details><summary>show</summary>
<p>

##### Overview

![03-05](https://user-images.githubusercontent.com/18049790/136649195-3178d098-badb-478f-8b16-3d7e1774a81f.png)

</p>
</details>

<details><summary>show</summary>
<p>

##### Prerequisites

```bash
clear
# Create the namespace
kubectl create namespace blue-green-namespace
```

```bash
clear
# Switch context into the namespace so that all subsequent commands execute inside that namespace.
kubectl config set-context --current --namespace=blue-green-namespace
```

</p>
</details>

<details><summary>show</summary>
<p>

##### Solution - Blue Deployment

```bash
clear
# Create the deployment as far as possible using the CLI (imperatively)
kubectl create deployment blue-deployment --image=nginx --replicas=10 --port=80 --dry-run=client -o yaml > ~/ckad/03-05-deploy-blue.yml
```

```bash
clear
# Edit the YAML file to make required changes
# Use the Question number in case you want to return to the question for reference or for review
vi ~/ckad/03-05-deploy-blue.yml
```

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: blue-deployment
  name: blue-deployment
spec:
  replicas: 10
  selector:
    matchLabels:
      app: blue-deployment
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: blue-deployment
        version: blue #👈👈👈 Add the label `version=blue`
        tier: web #👈👈👈 Add the label:  `tier=web`
    spec:
      containers:
      - image: docker.io/jamesbuckett/blue:latest
        name: blue
        ports:
        - containerPort: 80
        resources: {}
status: {}
```

```bash
clear
# Apply the YAML file to the Kubernetes API server
kubectl apply -f ~/ckad/03-05-deploy-blue.yml
```

```bash
clear
# Quick verification that the pod was created and is working
kubectl get pod --watch
```

```bash
clear
# Check labels
kubectl get pods -L tier -L version
```

</p>
</details>

<details><summary>show</summary>
<p>

##### Solution - Service

```bash
clear
# Create the namespace
kubectl expose deployment blue-deployment --port=80 --target-port=80 --name=bsg-service --dry-run=client -o yaml > ~/ckad/03-05-bsg-service.yml
```

```bash
clear
# Edit the YAML file to make required changes
vi ~/ckad/03-05-bsg-service.yml
```

```bash
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: blue-deployment
  name: bsg-service
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    #app: blue-deployment # Delete this
    tier: web #👈👈👈 Add the label:  `tier=web`. This is the sauce. One label pointing to both deployments
status:
  loadBalancer: {}
```

```bash
clear
# Apply the YAML file to the Kubernetes API server
kubectl apply -f ~/ckad/03-05-bsg-service.yml
```

</p>
</details>

<details><summary>show</summary>
<p>

##### Solution - Green Deployment

```bash
clear
# Create the deployment as far as possible using the CLI (imperatively)
kubectl create deployment green-deployment --image=nginx --replicas=10 --port=80 --dry-run=client -o yaml > ~/ckad/03-05-deploy-green.yml
```

```bash
clear
# Edit the YAML file to make required changes
# Use the Question number in case you want to return to the question for reference or for review
vi ~/ckad/03-05-deploy-green.yml
```

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: green-deployment
  name: green-deployment
spec:
  replicas: 10
  selector:
    matchLabels:
      app: green-deployment
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: green-deployment
        version: green #👈👈👈  Add the label `version=green`
        tier: web #👈👈👈 Add the label:  `tier=web`
    spec:
      containers:
      - image: docker.io/jamesbuckett/green:latest
        name: green
        ports:
        - containerPort: 80
        resources: {}
status: {}
```

```bash
clear
# Apply the YAML file to the Kubernetes API server
kubectl apply -f ~/ckad/03-05-deploy-green.yml
```

```bash
clear
# Quick verification that the pod was created and is working
kubectl get pod --watch
```

```bash
clear
# Check labels
kubectl get pods -L tier -L version
```

</p>
</details>

<details><summary>show</summary>
<p>

##### Solution - Green=70 & Blue=30

```bash
clear
# Scale Green to 7=70%
kubectl scale --replicas=7 deployment blue-deployment
```

```bash
clear
# Scale Blue to 3=30%
kubectl scale --replicas=3 deployment green-deployment
```

```bash
clear
# Check your work - 7 blue and 3 green
kubectl get pods -L tier -L version
```

Output:

```bash
NAME                                READY   STATUS    RESTARTS   AGE     VERSION   TIER
blue-deployment-5f855f68d6-295xb    1/1     Running   0          9m52s   blue      web
blue-deployment-5f855f68d6-2b4wv    1/1     Running   0          9m52s   blue      web
blue-deployment-5f855f68d6-2c9wn    1/1     Running   0          9m52s   blue      web
blue-deployment-5f855f68d6-5d4kb    1/1     Running   0          9m52s   blue      web
blue-deployment-5f855f68d6-7tqx7    1/1     Running   0          9m52s   blue      web
blue-deployment-5f855f68d6-k4s4w    1/1     Running   0          9m52s   blue      web
blue-deployment-5f855f68d6-vqv8m    1/1     Running   0          9m52s   blue      web
green-deployment-5b9f998d46-dlmsx   1/1     Running   0          112s    green     web
green-deployment-5b9f998d46-k6lpt   1/1     Running   0          112s    green     web
green-deployment-5b9f998d46-tjxl6   1/1     Running   0          112s    green     web
```

```bash
clear
# Check your work - curl the service to verify operation
kubectl run remote-run --image=busybox --restart=Never --rm -it
# Repeat this command to see different responses
wget -qO- bsg-service
```

Output

```bash
/ # wget -qO- bsg-service
Green !!!
Green !!!
Green !!!
/ # wget -qO- bsg-service
Blue !!!
Blue !!!
Blue !!!
/ # wget -qO- bsg-service
Blue !!!
Blue !!!
Blue !!!
/ # wget -qO- bsg-service
Blue !!!
Blue !!!
Blue !!!
/ # wget -qO- bsg-service
Blue !!!
Blue !!!
Blue !!!
/ # wget -qO- bsg-service
Blue !!!
Blue !!!
Blue !!!
/ # wget -qO- bsg-service
Blue !!!
Blue !!!
Blue !!!
/ # wget -qO- bsg-service
Blue !!!
Blue !!!
Blue !!!
/ # wget -qO- bsg-service
Green !!!
Green !!!
Green !!!
```

</p>
</details>

#### Clean Up

<details><summary>show</summary>
<p>

```bash
yes | rm -R ~/ckad/
kubectl delete ns deployment-namespace --force
kubectl delete ns edit-namespace --force
kubectl delete ns wordpress-namespace --force
kubectl delete ns blue-green-namespace --force
```

</p>
</details>

_End of Section_
