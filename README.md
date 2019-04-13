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

**Kubernetes concepts to learn:**

- Authorization seems really detailed and I don't understand it. There's like 63 different service accounts on KB right now I've just gotten started with the default install.
- What are "namespaces" -- I notice that there are already quite a few that kubernetes creates for itself--I haven't deployed anything yet.

### Goals

- Create a micro-service based system for my personal blog with a few distinct microservices (auth, blogapi, blog frontend, nginx proxy) which will be defined via a `docker-compose` file, and deployed on a Kubernetes (stack? namespace? container? IDK what you're supposed to call it)

- I should be able to successfully model "scaling" this stack over more or less virtual machines on my local computer.

Alright, so here goes.

## Table of Contents

- [4/13/2019 -- Chapter 1: The Basic Kubernetes Tutorial](diary/chapter_1.md)
