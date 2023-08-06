# Kubernetes Deployment Guide

## Table of Contents

- [Chapter 1 - Introduction to Kubernetes Deployments](#chapter-1---introduction-to-kubernetes-deployments)
- [Chapter 2 - Setting Up a Kubernetes Cluster](#chapter-2---setting-up-a-kubernetes-cluster)
- [Chapter 3 - Creating Kubernetes Deployments](#chapter-3---creating-kubernetes-deployments)  
- [Chapter 4 - Managing Kubernetes Deployments](#chapter-4---managing-kubernetes-deployments)
- [Chapter 5 - Updating Kubernetes Deployments](#chapter-5---updating-kubernetes-deployments)
- [Chapter 6 - Advanced Deployment Features](#chapter-6---advanced-deployment-features)
- [Chapter 7 - Troubleshooting and Securing Deployments](#chapter-7---troubleshooting-and-securing-deployments)
- [Chapter 8 - Conclusion](#chapter-8---conclusion)

## Chapter 1 - Introduction to Kubernetes Deployments

Kubernetes has quickly become one of the most popular platforms for deploying containerized applications. At its core, Kubernetes (also known as K8s) is an open-source system that helps with automating deployment, scaling, and management of containerized apps.

Some key benefits of using Kubernetes include:

- Automated rollouts and rollbacks - Kubernetes allows you to deploy new versions of your application in a controlled and automated way. This makes updating apps easy without downtime. 

- Self-healing - Kubernetes restarts containers that fail, replaces containers on nodes that die, and kills containers that don't respond to health checks. This keeps your application running smoothly.

- Horizontal scaling - You can scale your application up and down to meet demand by simply changing the replica count. Kubernetes handles spreading pods across nodes.

- Service discovery and load balancing - Kubernetes groups sets of pods into services, which are discoverable inside the cluster. This makes connecting components easy.

- Config and secret management - Kubernetes lets you store and manage sensitive config information and credentials for your apps declaratively.

- Batch execution - In addition to long-running services, Kubernetes can manage your batch and CI workloads, replacing containers that fail.

### What is a Kubernetes Deployment?

A Kubernetes deployment represents a set of identical pods running a particular version of your application. A deployment specifies the desired state for pods, and the Kubernetes control plane works to maintain that state. 

Deployments make it easy to update your running app without downtime. When you create a new deployment, Kubernetes handles upgrading pods to the new version seamlessly behind the scenes. It also lets you roll back gracefully in case something goes wrong.

Some key advantages of using Kubernetes deployments:

- Deployments are declarative - you simply describe the desired state in a YAML file

- Kubernetes handles replicating and scaling pods  

- Rolling updates make it easy to deploy new versions

- Health checks ensure availability during updates

- Revision history tracks changes over time

- Rollbacks revert to previous versions easily

We'll cover all aspects of deployments in depth through the course of this guide. By the end, you'll understand how to use deployments to keep your apps running smoothly on Kubernetes.

## Chapter 2 - Setting Up a Kubernetes Cluster 

Before we can start creating and managing deployments, we need a Kubernetes cluster to deploy them on. There are several options for setting up a cluster - in this chapter we'll go through some popular choices.

### Local Kubernetes with Minikube

One of the easiest ways to get started with Kubernetes is by using Minikube. Minikube sets up a single-node Kubernetes cluster locally on your machine for development and testing.

To install Minikube:

1. Install a hypervisor like VirtualBox or HyperKit
2. Download and install the Minikube binary  
3. Run `minikube start`

This will start a local Kubernetes cluster with one node. Minikube sets up a Docker registry, kubectl config, and all necessary components. 

Once Minikube is running, you can interact with the cluster using kubectl. Try verifying it with `kubectl get nodes` - you should see the single n1 node.

### Managed Kubernetes with GKE  

For production environments, you'll want to use a managed Kubernetes provider like Google Kubernetes Engine (GKE). Managed Kubernetes removes the burden of maintaining and upgrading your clusters.

Here are the steps to create a GKE cluster:

1. Install the gcloud CLI and initialize gcloud
2. Choose a project ID and enable the Kubernetes Engine API 
3. Create a GKE cluster with `gcloud container clusters create [CLUSTER_NAME]`
4. Retrieve credentials with `gcloud container clusters get-credentials [CLUSTER_NAME]`

Now kubectl will be configured to talk to your new GKE cluster. Test it with `kubectl get nodes` to see the managed server nodes.

### Setting up Kubernetes Locally via Docker

You can also run Kubernetes locally using Docker and a tool like Kind. Here's a quick overview:  

1. Install Docker and Kind
2. Create a cluster with `kind create cluster` 
3. Set KUBECONFIG to talk to the new cluster
4. Verify with `kubectl cluster-info`

Kind allows fast local testing since it containers everything. But it's not suitable for production due to lack of HA, scaling and other features.

In the next chapter we'll start using our cluster to create and manage deployments!

## Chapter 3 - Creating Kubernetes Deployments

Now that we have a Kubernetes cluster running, we can start using deployments to run applications. In this chapter we'll cover the deployment specification and examples of creating them. 

### Deployment Specification

A Kubernetes deployment is defined using a deployment manifest file in YAML or JSON format. Here are some key parts of a deployment spec:

**Metadata** - Includes a name and labels that identify the deployment.

**Spec** - Defines the desired state of the deployment including replicas, pod template, selector, strategy, etc.

**Template** - A template for the pods that will be created including labels, containers, volumes, etc.

**Selector** - Used to determine which pods belong to the deployment. Should match pod labels. 

**Replicas** - The number of pod replicas desired. Kubernetes will maintain this many pods.

### Creating a Deployment

Let's create a simple nginx deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment  
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

We can create this deployment by running:  

```
kubectl apply -f nginx-deployment.yaml
```

Kubernetes will now create 3 nginx pod replicas and ensure they are always running. We can inspect the deployment with `kubectl describe deployment nginx-deployment` and view the pods using `kubectl get pods`.

In the next chapter we'll go over managing and updating deployments to keep our apps running smoothly.

## Chapter 4 - Managing Kubernetes Deployments

In the previous chapter we created a simple deployment. Now let's go over how to manage deployments to keep our applications running properly.

### Checking Deployment Status 

We can check the status of a deployment using `kubectl describe deployment [NAME]`. This will show details like:

- The deployment strategy being used
- The number of current replicas vs the desired number
- The labels and selectors used to identify pods 
- The containers and images for pods
- The events related to the deployment like rollout history

For example:

```
kubectl describe deployment nginx-deployment
```

### Updating Deployments

To update our application to a new version, we need to update the deployment pod template to point to a new image:

```yaml
apiVersion: apps/v1
kind: Deployment
... 
spec:
  template:  
    spec:
      containers:
      - name: nginx
        image: nginx:1.16.1 # updated image
```

We can then apply this new spec:

```  
kubectl apply -f nginx-deployment.yaml
``` 

By default Kubernetes uses a rolling update strategy to roll out the new pods one by one, avoiding downtime. 

### Checking Rollout History   

We can view the history of deployments with:

```
kubectl rollout history deployment nginx-deployment 
```

This will list all revisions and changes made to the deployment so we can track changes over time.

### Rolling Back Deployments

If an update fails, we may need to roll back the deployment to the previous stable version:

```
kubectl rollout undo deployment nginx-deployment
```

This will roll back the pods to the prior specification. The rollout history makes it easy to revert any changes.  

There are many other deployment management topics like scaling, probes, and strategies that we'll cover in later chapters.

## Chapter 5 - Updating Kubernetes Deployments

In previous chapters we created and managed simple deployments. Now let's go deeper into strategies for updating deployments to deploy new versions of your application.

### Rolling vs Recreate Update Strategy

There are two main strategies for updating Kubernetes deployments:

**Rolling Update**  

This is the default strategy. It slowly updates pods instance by instance to the new version. It provides availability throughout the update.

**Recreate**

This terminates all old pods at once before creating new ones. This results in downtime during the update.

To configure the update strategy, add a `strategy` section in the deployment spec:

```yaml 
strategy:
  type: RollingUpdate # or Recreate
```

### Controlling Rolling Updates 

For rolling updates, we can tune the speed and process:

```yaml
strategy:
  rollingUpdate:  
    maxSurge: 1
    maxUnavailable: 0
```

`maxSurge` controls how many extra pods can be created during the update. `maxUnavailable` controls how many pods can be down at once.  

We can also configure the interval between pod updates using `minReadySeconds`.

### Handling Failed Rollouts 

Sometimes a new version may fail to start properly. In this case, Kubernetes will stop the rollout part way through.

We can check if a rollout failed with:

```
kubectl rollout status deployment/nginx-deployment 
```

If it's stuck, we can undo to the previous version using: 

```
kubectl rollout undo deployment/nginx-deployment
```

This makes rollback easy if an update fails.

In the next chapter we'll cover some more advanced deployment features like canary testing.

## Chapter 6 - Advanced Deployment Features  

We've covered the basics of Kubernetes deployments - now let's look at some more advanced features to enhance deployment strategies.

### Canary Deployments

A canary deployment slowly rolls out a new version to a subset of users first. This lets you test a release with a small percentage of traffic before rolling it out further. 

Here is an example canary deployment:

```yaml  
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-canary
spec:
  replicas: 10
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:  
        app: myapp
        track: canary
    spec:
      containers:
      - name: myapp
        image: myapp:v2 
```

This will deploy 1 new canary pod alongside the stable version. We can later update the `maxSurge` to gradually roll it out further.

### Blue-Green Deployments 

Blue-green deploys reduce downtime by running two production versions side by side. Here's an example:

```yaml
# Blue deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-blue
spec:
  replicas: 10  
  template:
    metadata:
      labels:
        app: myapp
        color: blue
    spec:
      containers:
      - name: myapp
        image: myapp:v1
        
---
        
# Green deployment        
apiVersion: apps/v1 
kind: Deployment
metadata:
  name: myapp-green
spec:
  replicas: 10
  template:
    metadata:
      labels:
        app: myapp
        color: green
    spec:
      containers:
      - name: myapp
        image: myapp:v2
```

Once the new green version is ready, we shift traffic from blue to green.

### A/B Testing

A/B testing deploys multiple versions and splits traffic to evaluate new features. Traffic is shifted gradually from A to B using weights.

There are many other advanced strategies like ramped rollouts, traffic mirroring, and shadow deployments that Kubernetes supports.

In the next chapter we'll look at deployment troubleshooting and security.

## Chapter 7 - Troubleshooting and Securing Deployments 

As with any complex system, issues can arise with Kubernetes deployments. In this chapter we'll cover troubleshooting deployment issues and securing your deployments.

### Investigating Deployment Issues

If a deployment is having issues, the first step is gathering more information:

- Check deployment status with `kubectl describe deployment` 
- View events with `kubectl get events`
- Check pod status and logs with `kubectl describe pod` and `kubectl logs`
- Verify Kubernetes master and node logs for errors  

Common issues include:

- Image pull failures - verify registry credentials
- Unhealthy pods - fix liveness and readiness checks   
- Insufficient resources - increase CPU/memory limits
- Node problems - check for node failures or evictions

### Debugging Failed Rollouts 

For rollout issues, check the rollout status: 

```
kubectl rollout status deployment/myapp
```

Check rollout history to see when issues began:

```
kubectl rollout history deployment/myapp
```

You can also monitor rollouts in real-time with `kubectl rollout status -w`.  

If needed, rollback to the last known good version:

```
kubectl rollout undo deployment/myapp 
```

### Securing Deployments

To secure deployments:

- Use namespace isolation and Network Policies
- Create RBAC rules limiting pod access
- Run pods as non-root users 
- Use secrets for sensitive config data
- Scan images for vulnerabilities
- Enable pod security policies
- Monitor audit logs for malicious events

Properly securing Kubernetes is critical for reducing risk.

This concludes the guide on Kubernetes deployments! You should now have a deeper understanding of running and managing apps with deployments. 

## Chapter 8 - Conclusion

In this comprehensive guide, we covered everything you need to know to start using Kubernetes deployments to run applications in production.

We started with an introduction to Kubernetes and the benefits of using deployments for app management. We saw how to set up a Kubernetes cluster using options like Minikube, GKE, and Kind.

We learned how to create and manage simple deployments, update deployments to new versions, and configure advanced deployment strategies like canary and blue/green deployments. We also looked at important topics like monitoring, troubleshooting and securing deployments. 

Some key takeaways include:

- Deployments provide a declarative way to manage apps on Kubernetes

- Use rolling updates to deploy new versions with zero downtime  

- Strategies like canary testing allow gradual rollouts

- Always have a rollback plan in case of issues   

- Monitor deployments closely and set up alerts

- Follow Kubernetes security best practices

With this foundation, you should feel comfortable taking your applications to production on Kubernetes. Deployments are a powerful primitive that enable continuous delivery and scaling.

For more information on Kubernetes deployments and other topics, refer to the official documentation at kubernetes.io and join the community discussions on Slack. Thank you for following along with this deployment guide!
