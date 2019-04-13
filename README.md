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

## Chapter 1. What is Kubernetes?

> Kubernetes (K8s) is an open-source system for automating deployment, scaling, and management of containerized applications.

This looks good: [Learn Kubernetes Basics](https://kubernetes.io/docs/tutorials/kubernetes-basics/).

Alright, so having Docker and Kubernetes on my computer, I'm just going work through this whole tutorial inside the `kubernetes-tutorial` directory. I think figuring out Kubernetes on a basic level is probably a good start before I try to integrate `docker-compose` with it.

Two types of resources:

- A _master_ node that manages the "cluster"
- _Nodes_ which are workers that run application containers (aha, so a Kubernetes cluster maps to an array of worker nodes each of which run docker containers. How does Kubernetes scale these worker nodes then? Wouldn't each platform work completely differently? Like, deploying virtual machines on my local computer seems like a very different proposition than deploying additional nodes on say Digital Ocean or EC2. Are there like Kubernetes "deployment drivers" per platform or something?)

Each node:

- Has a Kubelet (lol) for communicating with master.
- has container tools for managing the container it hosts (e.g. docker)

> A Kubernetes cluster can be deployed on either physical or virtual machines.

Aha. So there are "deployment drivers" basically. Kubernetes has to know either:

- how to create a new virtual machine on my local computer (in this case, through VirtualBox and Minikube)
- how to create a new physical machine (i.e. through the digital ocean API or an EC2 client or whatever).

There's basically a one-to-one mapping between physical machines and Kubernetes nodes in production.

(On another note, I have about 70GB available on my machine currently...I hope that's enough to emulate a Kubernetes stack...)

#### Starting my first cluster

Running `minikube start`. Apparently for each machine it creates a VirtualBox VM with 2048MB of virtual memory and 20GB of disk space? Oh boy... I really hope the 20GB is just the minikube virtual machine and it will deploy new nodes inside that...

Ok so `kubectl` is the command line tool I'll use to manage my cluster. (My poor computer's fans are really going at it...)

##### Brief interlude: bug with kubectl version

Ok so I need to [create a test service account](https://github.com/kubernetes/dashboard/wiki/Creating-sample-user) to access my Kubernetes dashboard...I would how this would work in prod?

When I enable the dashboard, or run any `*.yml` file with `kubectl apply -f` I'm getting the error `error: SchemaError(io.k8s.api.apps.v1.ReplicaSetList): invalid object doesn't have additional properties`

Turns out Docker Desktop installed an old version of `kubectl` on my system. I had to update and replace with `1.14` as of writing. Once I created the test user (using `dashboard-adminuser.yml` and `create-cluter-role-binding.yml` I was able to access the dashboard successfully).

##### Ok back to the tutorial

Alright so just exploring the [dashboard](http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/overview?namespace=default) a little bit I'm seeing a lot of concepts that I don't understand and that I probably should. Espeically

- Authorization seems really detailed and I don't understand it. There's like 63 different service accounts on KB right now I've just gotten started with the default install.
- What are "namespaces" -- I notice that there are already quite a few that kubernetes creates for itself--I haven't deployed anything yet.

I'm going to add these topics to my "things to learn" list above and move on.

> Once you have a running Kubernetes cluster, you can deploy your containerized applications on top of it. To do so, you create a Kubernetes Deployment configuration. The Deployment instructs Kubernetes how to create and update instances of your application. Once you've created a Deployment, the Kubernetes master schedules mentioned application instances onto individual Nodes in the cluster.

## FAQ

- Now that I have this great microservice architecture how do I route requests to mydomain.com to the cluster and how does that all work?

Kubernetes handles its own internal DNS between services. Most services won't be connected to the outside internet at all. But you need one service as a frontend router which will handle "ingress" into your cluster. [Like so](https://kubernetes.io/docs/concepts/services-networking/ingress/). In order for an ingress to work, you must have an [ingress controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/). This isn't anything fancy. It's basically a glorified nginx server, or something. Just a reverse-proxy server configured for kubernetes. One really good option is [traefik](https://github.com/containous/traefik). An ingress controller has a publically accessible IP address. You create an A record to that IP address with whoever manages your DNS records, and everything proceeds normally.

Want to add load balancing? Tack a load balancer service onto your kubernetes cluster. Like on digital ocean: https://www.digitalocean.com/docs/kubernetes/how-to/add-load-balancers/. In this example, the digital ocean load balancer can automatically handle letsencrypt etc, and will have an IP address that you can point to with an A record. From what I understand, the load balancer would probably just point at your existing ingress.
