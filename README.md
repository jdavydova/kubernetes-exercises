#  10 - Container Orchestration with Kubernetes
Exercises for Module "Container Orchestration with Kubernetes"


Your company’s java-mysql application (found here - https://gitlab.com/twn-devops-bootcamp/latest/10-kubernetes/kubernetes-exercises) is running with docker-compose on a server. This application is used often internally and by your company clients too. You noticed that the server isn't very stable: Often a database container dies or the application itself, or docker daemon must be restarted. During this time people can't access the app!

So when this happens, the users write to you to tell you that the app is down and ask you to fix it. You SSH into the server, restart the containers with docker-compose and containers start again.

But this is annoying work, plus it doesn't look good for your company that your clients often can't access the app. So you want to make your application more reliable and highly available. You want to replicate both the database and the app, so if one container goes down, there is always a backup. Also you don't want to rely on a single server, but have multiple, in case 1 whole server goes down or gets rebooted etc.



So you look into different solutions and decide to use the container orchestration tool Kubernetes to solve the issue. For now you want to configure it and deploy your application manually, since it's a new tool and want to try it out manually before automating.



🔹 EXERCISE 1: Create a Kubernetes cluster
Create a Kubernetes cluster (Minikube or LKE)

Start a local Kubernetes cluster:

    minikube start

Verify cluster status:

    kubectl get nodes

🔹 EXERCISE 2: Deploy Mysql with 2 replicas
First of all, you want to deploy the mysql database.

Deploy Mysql database with 2 replicas and volumes for data persistence
To simplify the process you can use Helm for this.

NOTE: Bitnami have recently changed their repository structure and have moved their MySQL helm charts to the "bitnamilegacy" repository. Add the following to your values YAML file in order to use the correct Helm chart:

    yaml
    global:
      security:
        allowInsecureImages: True
    image:
      registry: docker.io
      tag: latest
      repository: bitnamilegacy/mysql 
  
To clone repository locally:

    git clone https://github.com/jdavydova/kubernetes-exercises.git

    git checkout feature/solutions

    cd k8s-deployment

Mysql Chart link:
https://github.com/bitnami/charts/tree/master/bitnami/mysql

Minikube:

    helm repo add bitnami https://charts.bitnami.com/bitnami
    helm install my-release -f mysql-chart-values-minikube.yaml

    helm install my-release bitnami/mysql -f mysql-chart-values-minikube.yaml

🧠 What This Command Does:

    helm install
Tells Helm to install an application into Kubernetes.

    my-release
This is the release name.
It’s the name Helm gives to this installation.
You could name it:
mysql
prod-db
database
anything

Helm tracks installations by release name.

    bitnami/mysql
This is the chart.
It means:
"Download the mysql chart from the bitnami repository and install it."

### Notice about terminology:
🔹 What is a Chart in Helm?

A Helm chart is a package of Kubernetes templates.
Think of it like:

📦 A blueprint for installing an application in Kubernetes.
🧠 Simple Definition
A chart contains:
Deployment / StatefulSet templates
Services
ConfigMaps
Secrets
PersistentVolumeClaims
Default configuration (values.yaml)
All packaged together.

🔹 What is a “Blueprint”?
A blueprint is a detailed plan or design that tells you how to build something.
In construction:
Blueprint = architectural drawing of a house
It shows where walls, doors, electricity, plumbing go
Builders follow the blueprint to build the house

🔹 In Kubernetes Context
A Helm chart is like that blueprint.
It contains instructions that say:
Create a Deployment
Create a Service
Create a Secret
Create a Persistent Volume
Set replicas to 2
Use this Docker image
Use these environment variables
So instead of manually writing 5–10 YAML files…
You use a chart, and Helm builds everything automatically

LKE (stands for Linode Kubernetes Engine):

    helm repo add bitnami https://charts.bitnami.com/bitnami
    helm install my-release -f mysql-chart-values-lke.yaml

NOTICE:

🔹 What is LKE?

LKE (Linode Kubernetes Engine) is a managed Kubernetes service provided by Linode (Akamai Cloud).

In simple words:

LKE is a Kubernetes cluster running in the cloud instead of on your laptop.

🔹 How is LKE different from Minikube?
Minikube	LKE
Runs locally	Runs in the cloud
Single node	Multiple nodes
For learning	For production
Runs on your computer	Runs on Linode servers
🔹 What does “managed Kubernetes” mean?

With LKE:
Linode manages:
The Control Plane (API server, scheduler)
etcd
Cluster availability
Kubernetes upgrades
You manage:
Pods
Deployment
Services
Ingress
ConfigMaps / Secrets
Application scaling


🔹EXERCISE 3: Deploy your Java Application with 2 replicas
Now you want to:

Deploy the Java application with 2 replicas - note: once you have pushed your image to docker hub, you will need to ensure you are using this image name and tag in your application's configuration

With docker-compose, you were setting env_vars on the server. In K8s there are separate components for that, so you want to:

Before this exercises I build and push image to docker hub with command:

    docker buildx build --platform linux/amd64 \
    -t juliadavydova/my-app:1.0.1 \
    --push .

![img.png](img.png)

Create ConfigMap and Secret with the correct values and reference them in the application deployment config file.

# Create my-registry-key secret to pull image
DOCKER_REGISTRY_SERVER=docker.io
DOCKER_USER=your dockerID, same as for `docker login`
DOCKER_EMAIL=your dockerhub email, same as for `docker login`
DOCKER_PASSWORD=your dockerhub pwd, same as for `docker login`

kubectl create secret docker-registry my-registry-key \
--docker-server=$DOCKER_REGISTRY_SERVER \
--docker-username=$DOCKER_USER \
--docker-password=$DOCKER_PASSWORD \
--docker-email=$DOCKER_EMAIL


# Again from k8s-deployment folder, execute following commands - ensuring you have added your docker image details to line 22 of java-app.yaml
kubectl apply -f db-secret.yaml
kubectl apply -f db-config.yaml
kubectl apply -f java-app.yaml



🔹 EXERCISE 4: Deploy phpmyadmin
As a next step you:

Deploy phpmyadmin to access Mysql UI.
For this deployment you just need 1 replica, since this is only for your own use, so it doesn't have to be Highly Available. A simple deployment.yaml file and internal service will be enough.



Now your application setup is running in the cluster, but you still need a proper way to access the application. Also, you don't want users to access the application using the IP address but instead to use a domain name. For that, you want to install Ingress controller in the cluster and configure ingress access for your application.


🔹 EXERCISE 5: Deploy Ingress Controller
Deploy Ingress Controller in the cluster - using Helm


🔹 EXERCISE 6: Create Ingress rule
Create an Ingress rule for your application’s access.
If you are using Minikube, the application must be accessible on my-java-app.com
For LKE, use the Linode node-balancer address - line 48 of the Index.html file for the application must be updated to include the address

🔹 EXERCISE 7: Port-forward for phpmyadmin
However, you don't want to expose phpmyadmin for security reasons. So you configure port-forwarding for the service to access on localhost, whenever you need it.

Configure port-forwarding for phpmyadmin

As the final step, you decide to create a helm chart for your Java application where all the configuration files are configurable. You can then tell developers how they can use it by setting all the chart values. This chart will be hosted in its own git repository.


🔹 EXERCISE 8: Create Helm Chart for Java App

All config files: service, deployment, ingress, configMap, secret, will be part of the chart
Create custom values file as an example for developers to use when deploying the application
Deploy the java application using the chart with helmfile
Host the chart in its own git repository


