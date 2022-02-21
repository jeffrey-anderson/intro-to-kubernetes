# Kubernetes

## Background

* [What is Kubernetes?](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/)
* [The History of Kubernetes on a Timeline](https://blog.risingstack.com/the-history-of-kubernetes/)
* [The History of Kubernetes & the Community Behind It](https://kubernetes.io/blog/2018/07/20/the-history-of-kubernetes-the-community-behind-it/)
* [Cloud Native Computing Foundation](https://www.cncf.io/)
* [Cluster Architecture](https://kubernetes.io/docs/concepts/architecture/)

## Getting started

* [Kubernetes Tools](https://kubernetes.io/docs/tasks/tools/)
* [Minikube getting started](https://minikube.sigs.k8s.io/docs/start/)

## Reference
* [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
* [https://kubernetes.io/](https://kubernetes.io/)


## In-class demos

* [Week 1](week1/Week1.md)
* [Week 2](week2/Week2.md)


## What is Kubernetes?

> Kubernetes is a portable, extensible, open-source platform for managing containerized workloads and services, that facilitates both declarative configuration and automation. It has a large, rapidly growing ecosystem. Kubernetes services, support, and tools are widely available.<sup>1</sup>

## History of Kubernetes

Google introduced the [Borg System](https://ai.google/research/pubs/pub43438) around 2003-2004. It started off as a small-scale project, with about 3-4 people initially in collaboration with a new version of Google’s new search engine. Borg was a large-scale internal cluster management system, which ran hundreds of thousands of jobs, from many thousands of different applications, across many clusters, each with up to tens of thousands of machines.

In 2013, Google [introduced the Omega cluster management system](https://cloudplatform.googleblog.com/2014/06/an-update-on-container-support-on-google-cloud-platform.html), a flexible, scalable scheduler for large compute clusters. In 2014, Google introduced Kubernetes as an [open source version of Borg](https://www.youtube.com/watch?v=0W49z8hVn0k)

On July 21, 2015, [Kubernetes v1.0](https://github.com/kubernetes/kubernetes) is released and Google partners with the Linux Foundation to form the [Cloud Native Computing Foundation (CNCF)](https://www.cncf.io/). The CNFC aims to build sustainable ecosystems and to foster a community around a constellation of high-quality projects that orchestrate containers as part of a microservices architecture.

In February of 2016, [Helm](https://helm.sh/) is released to make it easier to deploy and manage large scale applications.

For more, see:
* [The History of Kubernetes on a Timeline](https://blog.risingstack.com/the-history-of-kubernetes/)
* Kubernetes [release timeline](https://en.wikipedia.org/wiki/Kubernetes#Release_timeline) on Wikipedia.

## Benefits of using Kubernetes

Kubernetes provides you with:<sup>1</sup>

* __Service discovery and load balancing.__ Kubernetes can expose a container using the DNS name or using their own IP address. If traffic to a container is high, Kubernetes is able to load balance and distribute the network traffic so that the deployment is stable.
* __Storage orchestration.__ Kubernetes allows you to automatically mount a storage system of your choice, such as local storages, public cloud providers, and more.
* __Automated rollouts and rollbacks.__ You can describe the desired state for your deployed containers using Kubernetes, and it can change the actual state to the desired state at a controlled rate. For example, you can automate Kubernetes to create new containers for your deployment, remove existing containers and adopt all their resources to the new container.
* __Automatic bin packing.__ You provide Kubernetes with a cluster of nodes that it can use to run containerized tasks. You tell Kubernetes how much CPU and memory (RAM) each container needs. Kubernetes can fit containers onto your nodes to make the best use of your resources.
* __Self-healing.__ Kubernetes restarts containers that fail, replaces containers, kills containers that don't respond to your user-defined health check, and doesn't advertise them to clients until they are ready to serve.
* __Secret and configuration management.__ Kubernetes lets you store and manage sensitive information, such as passwords, OAuth tokens, and SSH keys. You can deploy and update secrets and application configuration without rebuilding your container images, and without exposing secrets in your stack configuration.

## Kubernetes Architecture

![Kubernetes architecture](images/kube-architecture.png)

## Node types

## The API server

## etcd

## The scheduler

## The controller manager

## Kube-proxy

## Kubelet

## Container runtime

The [Container Runtime Interface (CRI)](https://kubernetes.io/docs/concepts/architecture/cri/) is a plugin interface which enables the kubelet to use a wide variety of container runtimes, without having a need to recompile the cluster components.

You need a working container runtime on each Node in your cluster, so that the kubelet can launch Pods and their containers. The CRI is the main protocol for the communication between the kubelet and Container Runtime.

The Kubernetes Container Runtime Interface (CRI) defines the main [gRPC](https://grpc.io/) protocol for the communication between the cluster components kubelet and container runtime.



## Cloud Controller (not shown)

The [cloud-controller-manager](https://kubernetes.io/docs/concepts/architecture/cloud-controller/) is a Kubernetes control plane component that embeds cloud-specific control logic and lets you link your cluster into your cloud provider's API, and separates out the components that interact with that cloud platform from components that only interact with your cluster.


## References:
1. [What is Kubernetes?](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/)