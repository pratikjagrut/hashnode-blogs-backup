---
title: "Kubernetes: Deployments (Part-1)"
seoTitle: "Kubernetes Deployments Explained: Configuration, Management, and Scali"
seoDescription: "Discover the essentials of Kubernetes Deployments in this comprehensive guide. Learn how to configure, create, manage, and scale Deployments to ensure high"
datePublished: Mon Jul 15 2024 15:14:04 GMT+0000 (Coordinated Universal Time)
cuid: clyn4kjn6000008kzc5rr3o7b
slug: kubernetes-deployments-part-1
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1720500758403/39beff73-4ceb-45b2-bad0-429152628ef6.png
tags: deployment, kubernetes, devops, containers

---

## Introduction

In the previous blog, we explored ReplicaSets and their importance in maintaining multiple identical pods for high application availability. However, in Kubernetes, we typically don't create ReplicaSets directly. Instead, we create higher-level objects such as Deployments, DaemonSets, or StatefulSets, which in turn manage ReplicaSets for us. In this blog, we'll delve into Deployments

Deployments are a crucial higher-level resource in Kubernetes, designed to manage the deployment and scaling of ReplicaSets, ensuring applications are always running in the desired state. We describe the desired state in the deployment configuration, and then the Deployment controller works to make the current state match the desired state.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721055695619/0d4a935c-59ea-4327-b65d-73b091838e33.png align="center")

## Difference Between Deployments and ReplicaSets

Deployments and ReplicaSets have different roles in Kubernetes. A ReplicaSet ensures a specified number of pod replicas are always running, but it lacks advanced update features. Deployments, however, allow for easy updates and rollbacks for pods and ReplicaSets. They support rolling updates and rollbacks. While ReplicaSets keep the right number of pods running, Deployments help manage application lifecycles, updates, and scaling without downtime, making them the better choice for managing stateless applications in Kubernetes.

## Configuring a Deployment

Configuring a Deployment involves defining a YAML file that specifies the desired state of the application. This includes details such as the application's image, the number of replicas, any necessary secrets and ConfigMaps, and a strategy for updating the application, ensuring smooth transitions with minimal downtime. Once the configuration file is applied, the Deployment controller works to ensure that the actual state matches the desired state.

Below is a sample YAML configuration for a Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
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
        image: nginx:latest
        ports:
        - containerPort: 80
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
```

## Breakdown of the YAML File

When writing any object in Kubernetes, you need to include certain required fields: `apiVersion`, `kind`, `metadata`, and `spec`.

1. **apiVersion:**
    
    ```yaml
    apiVersion: apps/v1
    ```
    
    This field specifies the version of the Kubernetes API that your object adheres to, ensuring compatibility with your Kubernetes cluster. In this case, it uses `apps/v1`.
    
2. **kind:**
    
    ```yaml
    kind: Deployment
    ```
    
    This field defines the type of Kubernetes object being created. In our YAML file, it indicates that we are creating a `Deployment`.
    
3. **metadata:**
    
    ```yaml
    metadata:
      name: nginx-deployment
      labels:
        app: nginx
    ```
    
    This section provides essential information about the Deployment:
    
    * **name:** This uniquely identifies the Deployment within its namespace (`nginx-deployment`). This is the only required field in `metadata`.
        
    * **labels:** Key-value pairs used to organize and select resources (`app: nginx`).
        
4. **spec:**
    
    ```yaml
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
            image: nginx:latest
            ports:
            - containerPort: 80
      strategy:
        type: RollingUpdate
        rollingUpdate:
          maxUnavailable: 1
          maxSurge: 1
    ```
    
    The `spec` section defines the desired state of the Deployment, including its pods and their configurations:
    
    * **replicas:** Specifies the number of pod replicas that the Deployment should maintain (3 in this case).
        
    * **selector:** Defines how the Deployment identifies the pods it manages.
        
        * **matchLabels:** A set of key-value pairs used to match the pods (`app: nginx`). This should be the same as the labels in `template.metadata.labels`.
            
    * **template:** Describes the pod's configuration to be created.
        
        * **metadata:** Includes labels to be applied to the pods.
            
        * **spec:** Defines the pod's configuration.
            
            * **containers:** Lists the containers within the pod.
                
                * **name:** Identifies the container (`nginx`).
                    
                * **image:** Specifies the Docker image to use (`nginx:latest`).
                    
                * **ports:** Indicates which ports should be exposed by the container (`containerPort: 80`).
                    
    * **strategy:** Defines the strategy for updating the Deployment.
        
        * **type:** Specifies the update strategy. In this case, it is `RollingUpdate`, which updates pods in a rolling fashion with minimal downtime.
            
        * **rollingUpdate:** Specifies parameters for the rolling update strategy, such as `maxUnavailable` and `maxSurge`.
            

## Creating a Deployment

To create a Deployment using the above YAML configuration, save the configuration to a file named `nginx-deployment.yaml` and apply it to the Kubernetes cluster using the following command:

```bash
kubectl apply -f nginx-deployment.yaml
```

## Managing a Deployment

You can list the Deployments using the `kubectl get deployments` command:

```bash
kubectl get deployments
```

Output:

```bash
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           10s
```

You can use the `kubectl describe` command to check the state of the Deployment:

```bash
❯ kubectl describe deployments.apps/nginx-deployment
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Mon, 15 Jul 2024 20:04:25 +0530
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=nginx
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  1 max unavailable, 1 max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:latest
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deployment-57d84f57dc (3/3 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  54s   deployment-controller  Scaled up replica set nginx-deployment-57d84f57dc to 3
```

List the ReplicaSets:

```bash
❯ kubectl get rs
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-57d84f57dc   3         3         3       93s
```

List the pods:

```bash
❯ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-57d84f57dc-w5n9r   1/1     Running   0          2m11s
nginx-deployment-57d84f57dc-8mf7x   1/1     Running   0          2m11s
nginx-deployment-57d84f57dc-gv24t   1/1     Running   0          2m11s
```

### Scaling the Deployment

You can scale the Deployment to a different number of replicas using the `kubectl scale` command:

```bash
❯ kubectl scale --replicas=5 deployment nginx-deployment
deployment.apps/nginx-deployment scaled
```

```bash
❯ kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   5/5     5            5           4m11s
```

```bash
❯ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-57d84f57dc-w5n9r   1/1     Running   0          4m24s
nginx-deployment-57d84f57dc-8mf7x   1/1     Running   0          4m24s
nginx-deployment-57d84f57dc-gv24t   1/1     Running   0          4m24s
nginx-deployment-57d84f57dc-9gjph   1/1     Running   0          27s
nginx-deployment-57d84f57dc-t47qd   1/1     Running   0          27s
```

### Deleting the Deployment

To delete the Deployment, use the following command:

```bash
kubectl delete deployment nginx-deployment
```

Deleting the Deployment will terminate all the pods it manages. If you want to keep the pods running after deleting the Deployment, use the `--cascade=orphan` flag:

```bash
kubectl delete deployment nginx-deployment --cascade=orphan
```

### Conclusion

In summary, Deployments play a crucial role in managing the desired state of your applications by ensuring a specified number of pod replicas are running at all times. This not only enhances the availability and reliability of your applications but also simplifies the management of pods in a Kubernetes environment.

By defining a Deployment through a YAML file, you can easily control the number of replicas, monitor their status, and scale them as needed, ensuring your applications remain resilient and performant.

***Thank you for reading this blog; your interest is greatly appreciated. I hope this information helps you on your Kubernetes journey. In the next part, we'll explore updating and rolling back Kubernetes deployments.***