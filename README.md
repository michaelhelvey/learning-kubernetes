# Microservice architecture + Docker + Kubernetes.

## Introduction - A diary

I already have some experience creating stacks of microservices with docker-compose and deploying them on single machines by just ssh'ing into the machine and running `docker-compose up`. But I know I still have a long way to go before I really understand how to deploy a cluster of docker container on Kubernetes (or docker swarm) and scale my microservices over time and load.

### Notes

This is not a tutorial. It's a developer's diary as I try to work through some concepts, in the hope that going back and overviewing my thought processes will be helpful to me and maybe to someone else.

### Questions

Moving forward, I want to answer a couple of questions:

- What really is Kubernetes?

- What does `docker stack deploy` do? I know I can deploy a compose file in `swarm` mode, but I don't really understand how that relates to virtual machines tbh.

- What is the difference between Kubernetes and Docker swarm?

- How can I model a multi-machine microservice architecture locally with docker? I have Kubernetes installed on my machine and loads of docs about using using Kubernetes locally with Docker to test your cluster, but no clue how to use it.

### Goals

- Create a micro-service based system for my personal blog with a few distinct microservices (auth, blogapi, blog frontend, nginx proxy) which will be defined via a `docker-compose` file, and deployed on a Kubernetes (stack? namespace? container? IDK what you're supposed to call it)

- I should be able to successfully model "scaling" this stack over more or less virtual machines on my local computer.

Alright, so here goes.

## Chapter 1. What is Kubernetes?

> Kubernetes (K8s) is an open-source system for automating deployment, scaling, and management of containerized applications.

This looks good: [Learn Kubernetes Basics](https://kubernetes.io/docs/tutorials/kubernetes-basics/).

Alright, so having Docker and Kubernetes on my computer, I'm just going work through this whole tutorial inside the `kubernetes-tutorial` directory.

Two types of resource:

- A _master_ node that manages the "cluster"
- _Nodes_ which are workers that run application containers (aha, so a Kubernetes cluster maps to an array of worker nodes each of which run docker containers. How does Kubernetes scale these worker nodes then? Wouldn't each platform work completely differently? Like, deploying virtual machines on my local computer seems like a very different proposition than deploying additional nodes on say Digital Ocean or EC2. Are there like Kubernetes "deployment drivers" per platform or something?)
