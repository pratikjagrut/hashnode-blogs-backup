---
title: "Kubernetes: Hello World"
seoTitle: "Beginner's Guide to Deploying Your First Application on Kubernetes"
seoDescription: "Explore the step-by-step process of deploying the first application on Kubernetes with this beginner's guide."
datePublished: Sun Jun 09 2024 09:46:30 GMT+0000 (Coordinated Universal Time)
cuid: clx7d0mbt000209l5cn0ehna6
slug: kubernetes-hello-world
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/9cXMJHaViTM/upload/8891bd64e2886c8b908fdfe2cccd28a5.jpeg
tags: kubernetes, devops, containers, k8s

---

### Introduction

Deploying software can be a daunting and unpredictable task. Kubernetes, often referred to as ***K8s***, serves as a proficient navigator in this complex landscape. It is an open-source container orchestration platform, that automates the deployment, scaling, and management of applications within containers. These containers are compact, self-sufficient units that house everything an application needs, ensuring consistency across diverse environments.

### Prepare the application

**Clone the Repository**

In this guide, we're using [***hello-Kubernetes***](https://github.com/pratikjagrut/hello-kubernetes)***, a*** simple web-based application written in Go. You can find the source code [here](https://github.com/pratikjagrut/hello-kubernetes).

```bash
git clone https://github.com/pratikjagrut/hello-kubernetes.git
cd hello-kubernetes
```

**Understanding the Code**

```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"os"
)

func handler(w http.ResponseWriter, r *http.Request) {
	log.Printf("Received request from %s", r.RemoteAddr)
	fmt.Fprintf(w, "Hello, Kubernetes!")
}

func main() {
	port := os.Getenv("PORT")
	if port == "" {
		port = "8080"
	}

	http.HandleFunc("/", handler)

	go func() {
		log.Printf("Server listening on port %s...", port)
		err := http.ListenAndServe(":"+port, nil)
		if err != nil {
			log.Fatal("Failed to start the server")
		}
	}()

	log.Printf("Click on http://localhost:%s", port)

	done := make(chan bool)
	<-done
}
```

This Go application sets up an HTTP server that responds with "Hello, Kubernetes!" and logs request details. It runs the server concurrently, keeping the main function active.

**The Dockerfile**

The Dockerfile uses a multi-stage build to create a minimal container:

* Builds the Go application.
    
* Creates a minimal final image using ***scratch***.
    
* Exposes port ***8080*** and sets the command to run the Go application.
    

```dockerfile
# Builder Stage
FROM cgr.dev/chainguard/go:latest as builder
# Set the working directory inside the container
WORKDIR /app
# Copy the application source code into the container
COPY . .
# Download dependencies using Go modules
RUN go mod download
# Build the Go application
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .
# Final Stage
FROM scratch
# Copy the compiled application binary from the builder stage to the final image
COPY --from=builder /app/main /app/main
# Expose port 8080 to the outside world
EXPOSE 8080
# Command to run the executable
CMD ["/app/main"]
```

**Building the Container Image**

1. **Docker**: Install Docker to create container images for your application. Refer to the official [Docker documentation](https://docs.docker.com/get-docker/) for installation instructions.
    
2. **Image Registry Account**: Sign up for an account on [GitHub](https://github.com), [DockerHub](https://hub.docker.com), or any other container image registry. You'll use this account to store and manage your container images.
    
3. Open the terminal and navigate to the repository directory.
    
4. Build the container image using the following command:
    
    ```bash
    docker build -t ghcr.io/pratikjagrut/hello-kubernetes .
    ```
    
    This command builds the container image using the `Dockerfile` current directory. The `-t` flag specifies the image name.
    

**Testing application image**

1. Once the image is built, run a Docker container from the image:
    
    ```bash
    ➜ docker run -p 8080:8080 ghcr.io/pratikjagrut/hello-kubernetes
    2023/08/08 13:25:24 Click on the link http://localhost:8080
    2023/08/08 13:25:24 Server listening on port 8080...
    ```
    
    This command maps port 8080 of your host machine to port 8080 in the container.
    
2. Open a web browser and navigate to [`http://localhost:8080`](http://localhost:8080). You should see the `Hello, Kubernetes!` message.
    

**Pushing the image to the container registry**

Here we've opted for the GitHub container registry. However, feel free to select a registry that aligns with your preferences.

1. Log in to Docker using the GitHub Container Registry:
    
    ```bash
    docker login ghcr.io
    ```
    
    When you run the command, it will ask for your username and password. Enter these credentials to log into your container registry.
    
2. Push the tagged image to the GitHub Container Registry:
    
    ```bash
    docker push ghcr.io/pratikjagrut/hello-kubernetes
    ```
    
3. Verify that the image is in your GitHub Container Registry by visiting the `Packages` section of your GitHub repository.
    

Next, we'll set up a Kubernetes cluster to deploy our containerized application.

### Setup Kubernetes cluster

Here we'll use KIND (Kubernetes in Docker) as our local k8s cluster.

**Installing KIND and Kubectl**

Before we dive into setting up the Kubernetes cluster, you'll need to install both KIND and kubectl on your machine.

* **KIND (Kubernetes in Docker)**: KIND allows you to run Kubernetes clusters as Docker containers, making it perfect for local development. Follow the [official KIND installation guide](https://kind.sigs.k8s.io/docs/user/quick-start/) to install it on your system.
    
* **kubectl**: This command-line tool is essential for interacting with your Kubernetes cluster. Follow the [Kubernetes documentation](https://kubernetes.io/docs/tasks/tools/) to install kubectl on your machine.
    

**Creating Your KIND Cluster**

Once KIND and Kubectl are set up, let's create your local Kubernetes cluster:

1. Open your terminal.
    
2. Run the following command to create a basic KIND cluster:
    
    ```bash
    kind create cluster
    ```
    
3. Check if the cluster is properly up and running using `kubectl get ns`
    
    It should get all the namespaces present in the cluster.
    
    ```bash
    ➜ kubectl get ns
    NAME                 STATUS   AGE
    default              Active   3m13s
    kube-node-lease      Active   3m14s
    kube-public          Active   3m14s
    kube-system          Active   3m14s
    local-path-storage   Active   3m9s
    ```
    

**Alternative Setup Options:**

* **Minikube**: If you prefer another local option, [Minikube](https://minikube.sigs.k8s.io/docs/start/) provides a hassle-free way to run a single-node Kubernetes cluster on your local machine.
    
* **Docker Desktop**: For macOS and Windows users, [Docker Desktop](https://www.docker.com/products/docker-desktop) offers a simple way to set up a Kubernetes cluster.
    
* **Rancher Desktop**: [Rancher Desktop](https://rancherdesktop.io/) is another choice for a local development cluster that integrates with Kubernetes, Docker, and other tools.
    
* **Cloud Clusters**: If you'd rather work in a cloud environment, consider platforms like [Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine) or [Amazon EKS](https://aws.amazon.com/eks/) for managed Kubernetes clusters.
    

With your Kubernetes cluster up and running, you're ready to sail ahead with deploying your first application.

### Deploy application on Kubernetes

Now, we'll deploy our application onto the Kubernetes cluster.

**Create a Kubernetes Deployment**

A **Deployment** in Kubernetes serves as a manager for your application's components, known as *Pods*. Think of it like a supervisor ensuring that the right number of Pods are running and matching your desired configuration.

In more technical terms, a Deployment lets you define how many Pods you want and how they should be set up. If a Pod fails or needs an update, the Deployment Controller steps in to replace it. This ensures that your application remains available and runs smoothly.

To put it simply, a Deployment takes care of keeping our application consistent and reliable, even when Pods face issues. It's a fundamental tool for maintaining the health of your application in a Kubernetes cluster.

Here's how we can create a Deployment for our application:

1. Create a YAML file named `app-deployment.yaml`:
    

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-k8s-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello-k8s
  template:
    metadata:
      labels:
        app: hello-k8s
    spec:
      containers:
        - name: hello-k8s-container
          image: ghcr.io/pratikjagrut/hello-kubernetes
          ports:
            - containerPort: 8080
```

This YAML defines a Deployment named `hello-k8s-deployment` that runs two replicas of our application.

1. Apply the Deployment to your Kubernetes cluster:
    

```bash
kubectl apply -f hello-k8s-deployment.yaml
```

Now, if you're using a GitHub registry just like me then you'll see an error(`ImagePullBackOff or ErrImagePull`)(`failed to authorize: failed to fetch anonymous token: unexpected status: 401 Unauthorized`) in deploying your application. By default the images on the GitHub container registry are private.

When you describe the pods you'll see warning messages in the events section such as `Failed to pull image "`[`ghcr.io/pratikjagrut/hello-kubernetes`](http://ghcr.io/pratikjagrut/hello-kubernetes)`"`.

```bash
➜ kubectl describe pods hello-k8s-deployment-54889c9777-549rn
...
Events:
  Type     Reason     Age                  From               Message
  ----     ------     ----                 ----               -------
  Normal   Scheduled  2m40s                default-scheduler  Successfully assigned default/hello-k8s-deployment-54889c9777-549rn to kind-control-plane
  Normal   Pulling    75s (x4 over 2m39s)  kubelet            Pulling image "ghcr.io/pratikjagrut/hello-kubernetes"
  Warning  Failed     74s (x4 over 2m39s)  kubelet            Failed to pull image "ghcr.io/pratikjagrut/hello-kubernetes": rpc error: code = Unknown desc = failed to pull and unpack image "ghcr.io/pratikjagrut/hello-kubernetes:latest": failed to resolve reference "ghcr.io/pratikjagrut/hello-kubernetes:latest": failed to authorize: failed to fetch anonymous token: unexpected status: 401 Unauthorized
  Warning  Failed     74s (x4 over 2m39s)  kubelet            Error: ErrImagePull
  Warning  Failed     50s (x6 over 2m39s)  kubelet            Error: ImagePullBackOff
  Normal   BackOff    36s (x7 over 2m39s)  kubelet            Back-off pulling image "ghcr.io/pratikjagrut/hello-kubernetes"
```

This happened because Kubernetes is trying to pull the private image and it does not have permission to do so.

When a container image is hosted in a private registry, we need to provide Kubernetes with credentials to pull the image via **Image Pull Secrets**.

1. Create a Docker registry secret:
    

```bash
kubectl create secret docker-registry my-registry-secret \
  --docker-username=<your-username> \
  --docker-password=<your-password> \
  --docker-server=<your-registry-server>
```

1. Attach the secret to your Deployment:
    

```yaml
spec:
  template:
    spec:
      imagePullSecrets:
        - name: my-registry-secret
```

Apply the changes::

```bash
kubectl apply -f hello-k8s-deployment.yaml
```

After applying the updated deployment you can see that all the pods are running.

```bash
➜ kubectl get pods
NAME                                    READY   STATUS    RESTARTS   AGE
hello-k8s-deployment-669788ccd6-4dbb6   1/1     Running   0          22s
hello-k8s-deployment-669788ccd6-k5gfg   1/1     Running   0          37s
```

**Access Your Application**

With the Deployment in place, we can access our application externally. Since we're using KIND, we can use port-forwarding to access the application:

1. Find the name of one of the deployed Pods:
    

```bash
kubectl get pods -l app=hello-k8s
```

1. Forward local port 8080 to the Pod:
    

```bash
kubectl port-forward <pod-name> 8080:8080
```

Now, if you open a web browser and navigate to [`http://localhost:8080`](http://localhost:8080) or use `curl http://localhost:8080` you should see "Hello, Kubernetes!" displayed, indicating your application is running successfully.

```bash
➜ curl http://localhost:8080
Hello, Kubernetes!%
```

> NOTE: For production, use a Kubernetes service and Ingress for optimal traffic handling.

### **Conclusion**

In conclusion, this beginner's guide has walked you through deploying your first application on Kubernetes. But remember, this is just the start. Kubernetes offers vast opportunities for optimizing your application's performance, scalability, and resilience. With features like advanced networking, load balancing, automated scaling, and self-healing, Kubernetes ensures seamless application operation in any environment. So, while this guide ends here, your journey with Kubernetes is only beginning.

Thank you for reading! Hope you find this helpful!