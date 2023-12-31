# Deploy-to-Kubernetes
> Deploy to Kubernetes in Google Cloud: Challenge Lab

Simple project demonstrates how to deploy a containerized application to kubernetes hosted on GCP



# Task 1. Create a Docker image and store the Dockerfile | Done


## Publish 

Push docker image to private artifact registry

Now you're going to push your image to the Google Artifact Registry. After that you'll remove all containers and images to simulate a fresh environment, and then pull and run your containers. This will demonstrate the portability of Docker containers.

To push images to your private registry hosted by Artifact Registry, you need to tag the images with a registry name. The format is <regional-repository>-docker.pkg.dev/my-project/my-repo/my-image.

## Create the target Docker repository

## Configure authentication 

Use these commands with the Docker client to pull the image. To use these commands, your Docker client must be configured to authenticate with us-central1-docker.pkg.dev. If this is the first time that you are pulling an image from us-central1-docker.pkg.dev with your Docker client, run the following command on the machine where Docker is installed.

```shell
gcloud auth configure-docker us-central1-docker.pkg.dev
```

## Push the container to Artifact Registry

+ Set Project ID 
`export PROJECT_ID=$(gcloud config get-value project)`

+ Build a new image's version 

```shell
docker build -t us-central1-docker.pkg.dev/$PROJECT_ID/thoi-repo/thoi-app:0.2 .
```

+ Push this image to Artifact Registry.

```shell
docker push us-central1-docker.pkg.dev/$PROJECT_ID/thoi-repo/thoi-app:0.2
```

## Test the image

1. Stop and remove all containers:

```shell
docker stop $(docker ps -q)
docker rm $(docker ps -aq)
```

2. Run the following command to remove all of the Docker images.

```shell
docker rmi us-central1-docker.pkg.dev/$PROJECT_ID/thoi-repo/thoi-app:0.2
docker rmi -f $(docker images -aq) # remove remaining images
docker images
```

3. Pull the image from GCR and run it.

```shell
docker pull us-central1-docker.pkg.dev/$PROJECT_ID/thoi-repo/thoi-app:0.2

docker run -p 4000:8000 -d us-central1-docker.pkg.dev/$PROJECT_ID/thoi-repo/thoi-app:0.2

curl http://localhost:4000
```

---
# Task 4. Create and expose a deployment in Kubernetes | In-progress

Create the <deployment.yaml> and <service.yaml> to deploy your new `container image` to a `Kubernetes cluster`.

1. Get the Kubernetes credentials before you deploy the image onto the Kubernetes cluster.

2. Before you create the deployments, make sure you check the deployment.yaml file

3. Create the deployments from the deployment.yaml and service.yaml files.

4. From the Navigation Menu, select Kubernetes Engine > Services & Ingress. Click on the load balancer IP Address of the deployed service to verify your services are up and running.

## Setup and Requirements 

* List the active account name 
 
    `gcloud auth list`

*  List the project ID
    
    `gcloud config list project`

## Set a default compute zone

1. Set the default compute region

    `gcloud config set compute/region us-central1`

2. Set the default compute zone

    `gcloud config set compute/zone us-central1-a`


## Create a GKE cluster

    gcloud container clusters create --machine-type=e2-medium --zone=us-central1-a thoi-cluster 


## Get authentication credentials for the cluster

`gcloud container clusters get-credentials lab-cluster `



## Deploy an application to the cluster

+ create a new Deployment

    `kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0`

+ Deployment Manifest 
    > Auto generated yaml deployment manifest by GKE engine

```yaml 
---
apiVersion: "apps/v1"
kind: "Deployment"
metadata:
  name: "thoi-server"
  namespace: "default"
  labels:
    thoi-app: "thoi-server"
spec:
  replicas: 3
  selector:
    matchLabels:
      thoi-app: "thoi-server"
  template:
    metadata:
      labels:
        thoi-app: "thoi-server"
    spec:
      containers:
      - name: "thoi-app-1"
        image: "us-central1-docker.pkg.dev/deploy-to-kubernetes-9999/thoi-repo/thoi-app:0.3"
---
apiVersion: "autoscaling/v2beta1"
kind: "HorizontalPodAutoscaler"
metadata:
  name: "thoi-server-hpa-3ikd"
  namespace: "default"
  labels:
    thoi-app: "thoi-server"
spec:
  scaleTargetRef:
    kind: "Deployment"
    name: "thoi-server"
    apiVersion: "apps/v1"
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: "Resource"
    resource:
      name: "cpu"
      targetAverageUtilization: 80

```

+ Container image

    `us-central1-docker.pkg.dev/deploy-to-kubernetes-9999/thoi-repo/thoi-app:0.3`

+ Create a Kubernetes Service, which is a Kubernetes resource that lets you expose your application to external traffic
    
    `kubectl expose deployment hello-server --type=LoadBalancer --port 8080`

## Deleting the cluster

    gcloud container clusters delete lab-cluster






