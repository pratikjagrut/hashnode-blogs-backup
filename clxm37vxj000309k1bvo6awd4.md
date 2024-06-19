---
title: "Introduction to Kubernetes"
seoTitle: "Kubernetes 101: Introduction, Key Features, and Architecture Explained"
seoDescription: "Discover the essentials of Kubernetes, the leading open-source platform for managing containerized applications. Learn about its features, architecture, and"
datePublished: Wed Jun 19 2024 17:08:45 GMT+0000 (Coordinated Universal Time)
cuid: clxm37vxj000309k1bvo6awd4
slug: introduction-to-kubernetes
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/9cXMJHaViTM/upload/fa86bb652ae693d05154d9b24af089c5.jpeg
tags: kubernetes, devops, containers, k8s, container-orchestration

---

## What is Kubernetes?

`Kubernetes is a portable, extensible, open-source platform for managing containerized workloads and services, that facilitates both declarative configuration and automation. It has a large, rapidly growing ecosystem. Kubernetes services, support, and tools are widely available.` - *From Kubernetes Docs*

In simpler terms, Kubernetes, also known as an orchestrator, is an open-source platform that automates the management of containers. Originally developed by `Google` as an internal container management project named `Borg`, Kubernetes was made open-source on `June 7, 2014`. In `July 2015`, it was donated to the `Cloud Native Computing Foundation (CNCF)`.

Kubernetes means `helmsman` or `pilot` in Greek, reflecting its role in guiding containerized applications. It is also known as `k8s`, a shorthand that represents the `8 letters between the 'k' and the 's'` .

## Key Features

Kubernetes offers a robust set of features, including:

* **Automatic Bin Packing:** Schedules containers automatically based on resource needs and constraints, ensuring efficient utilization without compromising availability.
    
* **Self-Healing:** Replaces and reschedules containers from failed nodes, restarts unresponsive containers, and prevents traffic from being routed to them.
    
* **Horizontal Scaling:** Scales applications manually or automatically based on CPU or custom metrics utilization.
    
* **Service Discovery and Load Balancing:** Assigns IP addresses to containers and provides a single DNS name for a set of containers to facilitate load balancing.
    
* **Automated Rollouts and Rollbacks:** Manages seamless rollouts and rollbacks of application updates and configuration changes, continuously monitoring application health to prevent downtime.
    
* **Secret and Config Management:** Manages secrets and configuration details separately from container images, avoiding the need to rebuild images.
    
* **Storage Orchestration:** Automatically mounts storage solutions to containers from local storage, cloud providers, or network storage systems.
    
* **Batch Execution:** Supports batch execution and long-running jobs, and replaces failed containers as needed.
    
* **Role-Based Access Control (RBAC):** Regulates access to cluster resources based on user roles within an enterprise.
    
* **Extensibility:** Extends functionality through Custom Resource Definitions (CRDs), operators, custom APIs, and more.
    

With its extensive array of capabilities, Kubernetes simplifies the management of containerized applications, ensuring optimal efficiency and performance, while also providing scalability, resilience, and flexibility for diverse workloads.

## Cluster Architecture

Kubernetes cluster has a straightforward yet elegant and efficient architecture, consisting of at least one `master node`, also known as a `control-plane node`, and at least one `worker node`. The diagram below illustrates the cluster architecture.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718725391854/d7b025b8-21df-44a7-9656-441eb93c0c33.png align="center")

### Control-plane node

The control plane is responsible for maintaining the overall state of the cluster. The control plane includes components like the API server, scheduler, controller manager, and etcd, which coordinate and manage the cluster operations.

**kube-api-server**

The Kubernetes API server serves as the core of the control plane. It exposes an HTTP API through which users, external components, and cluster components securely interact to manage the state of Kubernetes objects. It validates incoming requests before storing them and supports the incorporation of custom API servers to expand its functionality. Highly configurable, it accommodates diverse configurations and extensions to suit specific cluster requirements.

**Scheduler**

The Kubernetes scheduler watches for newly created pods without assigned nodes and selects suitable nodes for them. It retrieves resource usage data for each worker node from etcd via the API server. Additionally, it incorporates scheduling requirements specified in the pod's configuration, such as the preference to run on nodes labelled with specific attributes like "disk==ssd".

So basically, the scheduler is responsible for assigning a node to a pod based on available resources and scheduling constraints.

**kube-controller-manager**

The kube-controller-manager is a collection of different Kubernetes controllers, running as a single binary. It ensures that the `actual state of objects matches their desired state`. Each controller watches over its objects, maintains their state, and plays a specific role in maintaining the health and desired configuration of the cluster. Key controllers within kube-controller-manager include the `ReplicaSet controller, Deployment controller, Namespace controller, ServiceAccount controller, Endpoint controller, and Persistent Volume controller`.

**cloud-controller-manager**

The cloud-controller-manager includes the `Node controller, Route controller, Service controller, and Volume controller`. These controllers are responsible for interfacing with the cloud infrastructure to manage nodes, storage volumes, load balancing, and routing. They ensure seamless integration and management of cloud resources within the Kubernetes cluster.

**ETCD**

ETCD is a distributed key-value store used to persist the state of a Kubernetes cluster. New data is always appended, never replaced, and obsolete data is compacted periodically to reduce the data store size. Only the Kubernetes API server can communicate directly with ETCD to ensure consistency and security. The ETCD CLI management tool provides capabilities for backup, snapshot, and restore.

### Worker node

A worker node in Kubernetes executes application workloads by hosting and managing containers, providing essential computational resources within the cluster. It consists of components such as the `Kubelet`, `kube-proxy`, and a `container runtime interface(CRI)` like `Docker` or `containerd`.

**Kubelet**

The kubelet operates as an agent on each node within the Kubernetes cluster, maintaining communication with the control plane components. It receives pod definitions from the API server and coordinates with the container runtime to instantiate and manage containers associated with those pods. The kubelet also monitors the health and lifecycle of containers and manages resources as per pod specifications.

**kube-proxy**

The kube-proxy is a network agent running on each node in the Kubernetes cluster. It dynamically updates and maintains the network rules on the node to facilitate communication between pods and external traffic. It abstracts the complexities of pod networking by managing services and routing connections to the appropriate pods based on IP address and port number.

**Container Runtime**

In Kubernetes, a container runtime is essential for each node to handle the lifecycle of containers. Some of the known container runtimes are Docker, CRI-O, containerd, and rkt. These runtimes interface with Kubernetes using the Container Runtime Interface (CRI), ensuring that containers are created, managed, and terminated as needed within the cluster.

**Pod**

In Kubernetes, you cannot directly run containers as you would with Docker. Instead, containers are grouped into units called pods. A pod can host multiple containers and is the smallest deployable object in Kubernetes.

## How does all of this come together?

Let's see this with an example of pod creation:

First, define the pod by creating a YAML or JSON file that specifies its configuration, including `container images, resource requirements, environment variables, storage volumes etc`. This file acts as the pod's blueprint.

Once you have the pod definition file ready, you submit it to the Kubernetes API server using the `kubectl` command-line tool. For instance, you can apply the configuration with a command like `kubectl apply -f pod-definition.yaml`. Alternatively, you can create the pod directly with a command such as `kubectl run my-pod --image=my-image`.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718814000515/8a2a5de7-0ff0-4ed2-be16-be6b1227b1e1.png align="center")

The Kubernetes API server receives and validates the pod creation request, ensuring it meets all criteria. Once validated, it stores the pod definition in etcd, the cluster's key-value store.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718813953953/38ddc973-bad1-4256-a487-e776d3258157.png align="center")

The Kubernetes scheduler watches for newly created `pods without assigned nodes`. It interacts with the API server to get the pod configuration, evaluates the resource requirements and constraints, and then selects a suitable worker node for the pod, updating the pod configuration with the `nodeName`.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718813898885/ebea2757-0d78-4f8d-a46f-223697cbe6eb.png align="center")

On the assigned node, the kubelet receives the pod definition from the API server. `It manages pods and their containers, retrieves container images from a registry if needed, and uses the container runtime (like Docker or containerd) to create and start containers` as specified and `also set up storage as required`. It monitors container health and can restart them based on defined policies. Meanwhile, `kube-proxy configures networking rules for pod communication`. When all containers are running and ready, the pod can accept traffic, showcasing Kubernetes' orchestration in maintaining application states.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718815102888/d14984af-5e66-46aa-b961-0ef5b23da3bf.png align="center")

## Conclusion

Kubernetes is a robust platform for managing containerized applications, offering features that simplify deployment, scaling, and maintenance. This article covered its origins, core functionalities, and architecture. Leveraging Kubernetes enhances efficiency, resilience, and flexibility, making it essential for modern cloud-native environments.

***Thank you for taking the time to read this blog. Your interest and engagement are greatly appreciated. I hope this helps you on your Kubernetes journey. In the next article, we'll explore how to install Kubernetes using the Kubeadm tool.***