---
layout: post
title:  "Bashing through Minikube - 0 to 60 for Local Kubernetes Development"
date:   2018-08-20
tags: kubernetes minikube containers bash local development
author: Josh Armitage
image: img/kubernetes/minikube-logo.png
---

## Minikube You Say?
When you begin on a Kubernetes journey, often one of the first questions is; "how do I run locally"? That's where Minikube comes in; you get an easy to spin up single node Kubernetes environment where you can rapidly iterate. As with nearly all tools, there's some rough edges and gotchas to figure out. So in the next 5 minutes I hope to save you hours of frustration.

The blog was written using MacOS but the commands should translate across Linux distributions.


## Gotchas
#### 1. What `eval $(minikube docker-env)` Does
When you run Minikube it's running in a virtual machine, and in effect you are now running two parallel *docker environments*, the one on your local machine and the one in the virtual machine. What the `eval` command does is set a selection of environment variables such that your current terminal session is pointing to the *docker environment* in the virtual machine. This means all the images you have locally won't be available and vice versa.
#### 2. Changing Back To Your Local Docker Environment
The simplest way to change back is to create a new terminal session, however if you're in a script or want to continue to use your current session the magic runes to swap you back to your local docker environment is:

`unset DOCKER_API_VERSION DOCKER_TLS_VERIFY DOCKER_CERT_PATH DOCKER_API_VERSION DOCKER_HOST`
#### 3. Swapping Between Minikube And Other Clusters
Cluster autentication is managed via `config` files in `~/.kube/`. When you run `minikube start` it overwrites `~/.kube/config` by default, at the moment there is no command I have found for regenerating the config apart from starting the cluster again.
#### 4. Kubernetes Doesn't Come Fast Or Easily
The current thinking is becoming proficient with Kubernetes is a 3 to 6 month journey. If you're not planning on running Kubernetes in production you can get simple local container orchestration with `docker compose`.

## Patterns
###  1. Script Your Local Build And Local Run
With microservices you can reduce the context switch penalty by providing a common interface for every repository. I've found having the following 5 scripts in every repository sames time and many headaches:

1. `build.sh`
    This script builds the artifacts, i.e. this is where you call `mvn clean verify` or `npm run build`
    If you have a lot of artifacts generated from a repository it can be useful to make it so you can pass an artifact name to the script and build that in isolation for quicker feedback.
2. `local-build.sh`
    This script idempotently sets up the local environment, i.e. make sure minikube is running, make sure minikube has required dependent resources configured.
3. `local-run.sh`
    This script deploys Kubernetes manifests to Minikube, either via Helm, Kubectl or some other tooling. Also you can run integration tests here.
4. `local-nuke.sh`
    This script remove the resources from Kubernetes and deletes the Docker image artifact
5. `publish.sh`
    This script takes your artifacts and publishes them to your artifact repository.

This leads into the core of your continuous integration configuration being `build.sh && publish.sh`

### 2. Keep Your Containers Small
Using [multi-stage Docker builds](https://docs.docker.com/develop/develop-images/multistage-build/), i.e one container to build and one minimal container for runtime, will keep your feedback cycles fast. For example, with Java your `build` container can have the JDK installed with all your testing dependencies, but then you can create a JRE only Alpine container that can be about 5-10x smaller.

<div style="text-align: center; margin: 10px"><iframe width="560" height="315" src="https://www.youtube.com/embed/wGz_cbtCiEA?rel=0" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe></div>

### 3. Have Your Container Entrypoint And Command Be Targeted
The containers you are expecting to run in your cluster should be single purpose by design, this means that you can have your entrypoint and command set so you don't have to manage that outside the Dockerfile itself.

### 4. Exposing Ports
#### e.g. I want to see my postgres database
`kubectl port-forward $(kubectl get pods | grep db | awk {'print $1'}) 5432:5432`

This is an example command for accessing a Postgres database on the cluster. If we break it down:
1. `kubectl port-forward {pod-name} source-port:destination-port`

    This command sets up the port forwarding so the given pod port is accessible on `localhost:destination-port`
1. `kubectl get pods | grep db | aws {'print $1'}`

    This command lists all the pods, selects the line that has the term `db` and then prints the first column of data, i.e. the pod name.

### 5. Alias Away Common Command Chains
Minikube, alas, is not perfect which means you have to restart it from scratch on occasion. Also when iterating on the scripts from pattern #1 it's helpful to make sure they work on a fresh machine.

`alias reset-minikube='minikube delete && minikube start && helm init`

Also for iterating on your integration test feedback loop:

`alias warm-reload='./local-nuke.sh && ./local-build.sh && ./local-run.sh'`

And for managing the docker environments:

`alias local-docker='unset DOCKER_API_VERSION DOCKER_TLS_VERIFY DOCKER_CERT_PATH DOCKER_API_VERSION DOCKER_HOST'`

## Anti-Patterns
### 1. Every Project In A Repository Should Be Testable In Isolation
With Docker it is so trivial to spin up a database container that people can make the mistake of making all their component tests require an actual database to function. This complicates matters significantly especially when you add parallel docker environments between minikube and your local machine.

### 2. Don't Rely On Being Able To Run Your Entire Stack Locally
When you have a small number of services it is possible to run them all locally at once, however this stops being the case quickly and is more difficult to retroactively fix then to stop this at the start.

The need to do this is generally driven by insufficient testing, or testing at the wrong isolation levels. For testing across repositories leave that to Continuous Delivery.

## Wrap Up
Hopefully this post will have saved you many frustrations in the road ahead to Kubernetes adoption. Minikube is fantastic for keeping your feedback cycles fast and learning its quirks is well worth the time investment.

The next step is getting to grips with your Kubernetes flavour of choice, be that cloud vendor managed or for those stout of heart a bare metal installation. Stay tuned for posts on adopting [EKS](https://aws.amazon.com/eks/), up-skilling the team into Kubernetes, and making your cluster production ready.
