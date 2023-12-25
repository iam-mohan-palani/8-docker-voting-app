# Example Voting App

A simple distributed application running across multiple Docker containers.

## Steps
1.In any VM -> Install docker and clone this repo.

# a) Without docker compose
1.Build container images worker, voting , result using DOCKERFILE in repo and name them worker,voting-app,result-app.
2.Pull the docker images of redis and postgres:9.4
3.Run following commands to run the container images and link them.
  docker run -d --name=redis redis
  docker run -d --name=db postgres:9.4
  docker run -d --name=vote -p 5000:80 --link redis:redis voting-app
  docker run -d --name=result -p 5001=80 --link db:db result-app
  docker run -d --name=worker --link db:db --link redis:redis worker
4.View the voting app at PORT 5000 and result at PORT 5001.

# b) With docker compose version 2
1. Run
   sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   sudo chmod +x /usr/local/bin/docker-compose
2. (optional) Build container images worker, voting , result using DOCKERFILE in repo and name them worker,voting-app,result-app.
3.Create docker-compose.yml with below contents.
  
  redis:
      image:redis
  db:
      image:postgres:9.4
  vote:
     image:voting-app
     parts:
         -5000:80
     links:
         -redis
  result:
     image:result-app
     parts:
         -5001:80
     links:
         -db
  worker:
     image:worker-app
     links:
         -db
         -redis
4. RUN docker compose up
5. View the voting app at PORT 5000 and result at PORT 5001.

# c) With docker compose version 3
Refer -> https://docs.docker.com/compose/compose-file/compose-file-v3/
1. Run
   sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   sudo chmod +x /usr/local/bin/docker-compose
2. (optional) Build container images worker, voting , result using DOCKERFILE in repo and name them worker,voting-app,result-app.
3.Create docker-compose.yml with below contents.
version:"3"
services:  
  redis:
      image:redis
  db:
      image:postgres:9.4
  vote:
     image:voting-app
     parts:
         -5000:80
  result:
     image:result-app
     parts:
         -5001:80
  worker:
     image:worker-app
     
4. RUN docker compose up
5. View the voting app at PORT 5000 and result at PORT 5001.    

## Getting started

Download [Docker Desktop](https://www.docker.com/products/docker-desktop) for Mac or Windows. [Docker Compose](https://docs.docker.com/compose) will be automatically installed. On Linux, make sure you have the latest version of [Compose](https://docs.docker.com/compose/install/).

This solution uses Python, Node.js, .NET, with Redis for messaging and Postgres for storage.

Run in this directory to build and run the app:

```shell
docker compose up
```

The `vote` app will be running at [http://localhost:5000](http://localhost:5000), and the `results` will be at [http://localhost:5001](http://localhost:5001).

Alternately, if you want to run it on a [Docker Swarm](https://docs.docker.com/engine/swarm/), first make sure you have a swarm. If you don't, run:

```shell
docker swarm init
```

Once you have your swarm, in this directory run:

```shell
docker stack deploy --compose-file docker-stack.yml vote
```

## Run the app in Kubernetes

The folder k8s-specifications contains the YAML specifications of the Voting App's services.

Run the following command to create the deployments and services. Note it will create these resources in your current namespace (`default` if you haven't changed it.)

```shell
kubectl create -f k8s-specifications/
```

The `vote` web app is then available on port 31000 on each host of the cluster, the `result` web app is available on port 31001.

To remove them, run:

```shell
kubectl delete -f k8s-specifications/
```

## Architecture

![Architecture diagram](architecture.excalidraw.png)

* A front-end web app in [Python](/vote) which lets you vote between two options
* A [Redis](https://hub.docker.com/_/redis/) which collects new votes
* A [.NET](/worker/) worker which consumes votes and stores them in…
* A [Postgres](https://hub.docker.com/_/postgres/) database backed by a Docker volume
* A [Node.js](/result) web app which shows the results of the voting in real time

## Notes

The voting application only accepts one vote per client browser. It does not register additional votes if a vote has already been submitted from a client.

This isn't an example of a properly architected perfectly designed distributed app... it's just a simple
example of the various types of pieces and languages you might see (queues, persistent data, etc), and how to
deal with them in Docker at a basic level.
