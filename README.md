# Docker Workshop

This repository contains a small project used for an introduction to Docker and
related technologies at the MEDIAn boostcamp 2019 in New York.

## Docker Desktop

Check the docker version:
```
docker --version
```

Run your first container:

```
docker run hello-world
```

Run ubuntu:

```
docker run -it ubuntu bash
```

Where `-it` means interactive tty (text telephone).

List all locally available images:

```
docker image ls
```

## The Dockerfile

Build the app:
```
cd helloworld/
docker build -t friendlyhello .
```

Execute the app:
```
docker run -p 4000:80 friendlyhello
```

Test it:
```
curl http://localhost:4000
```

We see some warnings, so lets update the python version.

1. Edit the Dockerfile and change the base image to `python:3.7-slim`
1. Build the app again: `docker build -t friendlyhello .`
1. Execute the app again: `docker run -p 4000:80 friendlyhello`
1. Test it again: `curl http://localhost:4000`


## Create Services with Docker Compose

We have seen in the web interface that redis is missing.
The different parts of the distributed abblication are services.
For the application to be fully functional all services need to be deployed.

Take a look into `docker-compose.yaml` with your favorite text
editor then run the following two commands:

```
docker-compose build
docker-compose up
```

To hide the logs:
```
docker-compose up -d
docker-compose down
```

The deploy section in the compose files was ignored so far
because we were running the services locally using docker compose.

### Recap

We have seen how we can

- run an image
- write a Dockerfile
- build an image from it
- write a docker-compose file
- start all services defined in a compose file

## Docker Compose for Development

In this section we take a closer look into the Dockerfile and
how we can best use it for development.

Checkout the git repository:

```
git clone git@github.com:nicohein/predict-api.git
```

Take a look into the Dockerfile and Dockerfile.dev as well as
the docker-compose.yaml.

```
docker-compose up
```

Then change anything in e.g. /app/predictor/models.py and look for log:

```
Detected change in '/app/predictor/models.py', reloading
```

We have seen the advantageg of using volumes in development.
Now lets go back to the last project.

## Docker Swarm

Initialize docker Swarm

```
docker swarm init
```

```
docker node ls
```

### Docker Swarm Service

We now have a one node cluster and we can create a service on it.

```
docker service create --name friendlyhellosvc friendlyhello
docker service ls
docker service inspect friendlyhellosvc
```

This service can be scaled in an declarative way.

```
docker service scale friendlyhellosvc=10
docker service ls
```

Remove the service

```
docker service rm friendlyhellosvc
```

### Docker Swarm with Compose file

```
docker stack deploy --compose-file docker-compose.yaml friendlyhellostack
docker service ls
docker service logs friendlyhellostack_redis
```

## Create a Kubernetes Cluster

We do not code the infastructure (e.g. Terraform or Resource Templates) for the first steps but leverage the api:

```
export NAME=nico
```

1. Activate the right subscription:

```
az account list --refresh --output table
az account set -s "Docker introduction"
```

2. Create a Resource Group

```
az group create \
              --name=predictionapi-$NAME \
              --location=eastus \
              --output table
```

_Delete_

```
az group delete \
              --name=predictionapi-$NAME \
              --output table
```

3. Create a ssh-key

```
mkdir predictioncluster
cd predictioncluster

ssh-keygen -f ssh-key-predictioncluster
```

4. Create the cluster

```
az aks create --name predictioncluster-$NAME \
              --resource-group predictionapi-$NAME \
              --ssh-key-value ssh-key-predictioncluster.pub \
              --node-count 3 \
              --node-vm-size Standard_A2_v2 \
              --output table
```

5. Configure kubectl

```
az aks get-credentials \
             --name predictioncluster-$NAME \
             --resource-group predictionapi-$NAME \
             --output table
```

6. Check cluster functionality

```
kubectl get node
```



7. Create a container registry

```
az acr create --name prdictionapiacr \
              --resource-group predictionapi-$NAME \
              --sku Standard
```

8. Tag the predictionapi images - the `acrLoginServer` is part of the output from acr creation.

```
docker build -t prdictionapiacr.azurecr.io/predictionapi:0.0.1 .
az acr login --name prdictionapiacr

docker push prdictionapiacr.azurecr.io/predictionapi
```

9. Dploy configmap, service and application:

```
kubectl apply -f predictioncluster/configmaps.yaml
kubectl apply -f predictioncluster/deployments.yaml
kubectl apply -f predictioncluster/services.yaml
```

10. Delete the cluster

```
az aks delete --name predictioncluster-$NAME \
               --resource-group predictionapi-$NAME
```
