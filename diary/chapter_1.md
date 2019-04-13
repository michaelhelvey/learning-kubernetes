# Chapter 1. What is Kubernetes? (Basic Tutorial)

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

## Starting my first cluster

Running `minikube start`. Apparently for each machine it creates a VirtualBox VM with 2048MB of virtual memory and 20GB of disk space? Oh boy... I really hope the 20GB is just the minikube virtual machine and it will deploy new nodes inside that...

Ok so `kubectl` is the command line tool I'll use to manage my cluster. (My poor computer's fans are really going at it...)

#### Brief interlude: bug with kubectl version

Ok so I need to [create a test service account](https://github.com/kubernetes/dashboard/wiki/Creating-sample-user) to access my Kubernetes dashboard...I would how this would work in prod?

When I enable the dashboard, or run any `*.yml` file with `kubectl apply -f` I'm getting the error `error: SchemaError(io.k8s.api.apps.v1.ReplicaSetList): invalid object doesn't have additional properties`

Turns out Docker Desktop installed an old version of `kubectl` on my system. I had to update and replace with `1.14` as of writing. Once I created the test user (using `dashboard-adminuser.yml` and `create-cluter-role-binding.yml` I was able to access the dashboard successfully).

#### Ok back to the tutorial

Alright so just exploring the [dashboard](http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/overview?namespace=default) a little bit I'm seeing a lot of concepts that I don't understand and that I probably should. Espeically

- Authorization seems really detailed and I don't understand it. There's like 63 different service accounts on KB right now I've just gotten started with the default install.
- What are "namespaces" -- I notice that there are already quite a few that kubernetes creates for itself--I haven't deployed anything yet.

I'm going to add these topics to my "things to learn" list above and move on.

> Once you have a running Kubernetes cluster, you can deploy your containerized applications on top of it. To do so, you create a Kubernetes Deployment configuration. The Deployment instructs Kubernetes how to create and update instances of your application. Once you've created a Deployment, the Kubernetes master schedules mentioned application instances onto individual Nodes in the cluster.

I can create a deployment by calling `kubectl run <deployment-name> image=mydomain.com/path/to/docker-image/` When Kubernetes creates the deployment, it initializes a "pod" to host an instance of the running docker container.

> A Pod is a Kubernetes abstraction that represents a group of one or more application containers (such as Docker or rkt), and some shared resources for those containers.

> A Pod models an application-specific "logical host" and can contain different application containers which are relatively tightly coupled. For example, a Pod might include both the container with your Node.js app as well as a different container that feeds the data to be published by the Node.js webserver. The containers in a Pod share an IP Address and port space, are always co-located and co-scheduled, and run in a shared context on the same Node.

> Pods are the atomic unit on the Kubernetes platform. When we create a Deployment on Kubernetes, that Deployment creates Pods with containers inside them (as opposed to creating containers directly).

So as I understand it, Pods are basically an abstraction over containers which allow Kubernetes to

##### Container -> Pods -> Nodes -> -> Cluster (/ Service see below)

> A Pod always runs on a Node. A Node is a worker machine in Kubernetes and may be either a virtual or a physical machine, depending on the cluster. Each Node is managed by the Master. A Node can have multiple pods, and the Kubernetes master automatically handles scheduling the pods across the Nodes in the cluster.

##### Services

> A Service in Kubernetes is an abstraction which defines a logical set of Pods and a policy by which to access them. Services enable a loose coupling between dependent Pods. . . Although each Pod has a unique IP address, those IPs are not exposed outside the cluster without a Service.

Ok so this is the point where we connect the cloud to the outside world (potentially).

> A Service routes traffic across a set of Pods. Services are the abstraction that allow pods to die and replicate in Kubernetes without impacting your application. Discovery and routing among dependent Pods (such as the frontend and backend components in an application) is handled by Kubernetes Services.

### Conclusion: Development and deployment in a Kubectl heirarchy

- You create your application, and push it to a docker registry (you can host your own registry quite easily; it's just a Python app, which you can run like any other Docker Container) [Explore the docs](https://docs.docker.com/registry/deploying/).
- You create a Kubernetes Deployment in your cluster. Example: `kubectl run kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1` (You can automatically create a deployment in the same step with the `--expose` option. See below). A deployment will have a Pod to run it.
- You create a Service to describe a way to access a Pod or set of pods and expose it, either internally within the Kubernetes network, or on the external network.
- Your 'top of the tree' service will probably be some kind of reverse proxy that will proxy requests to your domain to whatever your frontend is, etc, and requests to whatever your user-facing APIs are. (Like consumer auth APIs, etc). For RRM's usecase, for example, every single service is user-facing one some level (users being either admins or customers). We don't really have "microservices" that are purely internal. (Maybe we should? Idk)
- You push updates by deploying a new version of your docker image (using tags like "image:v1" "image:v2" etc) and then deploying using rolling updates to Pods with `kubectl set image`. You can also rollback if there's any errors.

### Random Questions I've Found Out Along the Way During Chapter 1

1. **Now that I have this great microservice architecture how do I route requests to mydomain.com to the cluster and how does that all work?**

Kubernetes handles its own internal DNS between services. Most services won't be connected to the outside internet at all. But you need one service as a frontend router which will handle "ingress" into your cluster. [Like so](https://kubernetes.io/docs/concepts/services-networking/ingress/). In order for an ingress to work, you must have an [ingress controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/). This isn't anything fancy. It's basically a glorified nginx server, or something. Just a reverse-proxy server configured for kubernetes. One really good option is [traefik](https://github.com/containous/traefik). An ingress controller has a publically accessible IP address. You create an A record to that IP address with whoever manages your DNS records, and everything proceeds normally.

Want to add load balancing? Tack a load balancer service onto your kubernetes cluster. Like on digital ocean: https://www.digitalocean.com/docs/kubernetes/how-to/add-load-balancers/. In this example, the digital ocean load balancer can automatically handle letsencrypt etc, and will have an IP address that you can point to with an A record. From what I understand, the load balancer would probably just point at your existing ingress.

2. **[You can auto scale with Kubernetes!](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)**

3. **ELK stack**

You get ELK stack-like functionality for free as far as logging goes in Kubernetes by configuring your cluster to send all the node cluster logs to an internal elasticsearch / kibana stack running inside the cluster. You could access these pods by defining an ingress for them. Or just use `kubectl proxy` with bearer tokens or something.

[View Docs](https://kubernetes.io/docs/tasks/debug-application-cluster/logging-elasticsearch-kibana/)

This sounds great for basic application logging - basically you get Google Cloud + stackdriver for free. For more advanced security logs you'd probably want a custom stack (maybe a separate cluster? Not sure if that's something you really do) for `auditd` logs etc.
