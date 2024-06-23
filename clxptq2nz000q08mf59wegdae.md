---
title: "Creating a Kubernetes Cluster with Kubeadm and Containerd: A Comprehensive Step-by-Step Guide"
seoTitle: "Step-by-Step Guide to Creating a Kubernetes Cluster with Kubeadm"
seoDescription: "Learn how to create a Kubernetes cluster using Kubeadm with this comprehensive step-by-step guide. From prerequisites to setting up container runtime and in"
datePublished: Sat Jun 22 2024 07:54:02 GMT+0000 (Coordinated Universal Time)
cuid: clxptq2nz000q08mf59wegdae
slug: creating-a-kubernetes-cluster-with-kubeadm-and-containerd-a-comprehensive-step-by-step-guide
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1719153747285/23d49598-be16-4275-8a53-19052cd1bce8.png
tags: tutorial, kubernetes, containers, k8s, ckad, cka, cluster, kubeadm, step-by-step-guide

---

Kubeadm is a tool designed to simplify the process of creating Kubernetes clusters by providing `kubeadm init` and `kubeadm join` commands as best-practice "fast paths." - Kubernetes documentation

In this blog, we'll go through the step-by-step process of installing a Kubernetes cluster using Kubeadm.

## Prerequisites

Before you begin, ensure you have the following:

1. Ensure you have a compatible Linux host (e.g., Debian-based and Red Hat-based distributions). In this blog, we're using Ubuntu which is a Debian-based OS.
    
2. At least 2 GB of RAM and 2 CPUs per machine.
    
3. Full network connectivity between all machines in the cluster.
    
4. Unique hostname, MAC address, and product\_uuid for every node.
    
5. Ensure all the required ports are open for the control plane and the worker nodes. You can refer to [Ports and Protocols](https://kubernetes.io/docs/reference/networking/ports-and-protocols) or see the screenshot below.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718886897927/49b67a48-825f-4315-b336-71169fb5a644.png align="center")
    

***Disable swap on all the nodes.***

```bash
sudo swapoff -a
# disable swap on startup in /etc/fstab
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

## Setup container runtime(Containerd)

To run containers in Pods, Kubernetes uses a container runtime. By default, Kubernetes employs the `Container Runtime Interface (CRI)` to interact with your selected container runtime. Each node needs to have container runtime installed. In this blog, we'll use `containerd`.

***Run these instructions on all the nodes. I am using Ubuntu on all the nodes.***

* **Enable IPv4 packet forwarding:**
    
    ```bash
    # sysctl params required by setup, params persist across reboots
    cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
    net.ipv4.ip_forward = 1
    EOF
    
    # Apply sysctl params without reboot
    sudo sysctl --system
    ```
    
    Run `sudo sysctl net.ipv4.ip_forward` to verify that `net.ipv4.ip_forward` is set to 1
    
* **Specify and load the following kernel module dependencies:**
    
    ```bash
    cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
    overlay
    br_netfilter
    EOF
    
    sudo modprobe overlay
    sudo modprobe br_netfilter
    ```
    
* **Install containerd:**
    
    ***Add Docker's official GPG key:***
    
    ```bash
    sudo apt-get update
    sudo apt-get -y install ca-certificates curl
    sudo install -m 0755 -d /etc/apt/keyrings
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc
    ```
    
    ***Add the repository to Apt sources:***
    
    ```bash
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
      $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```
    
    ***Install containerd***
    
    ```bash
    sudo apt-get update
    sudo apt-get -y install containerd.io
    ```
    
    For more details, refer to the [Official installation documentation](https://github.com/containerd/containerd/blob/main/docs/getting-started.md#option-2-from-apt-get-or-dnf)
    
* **Configure systemd cgroup driver for containerd**
    
    First, we need to create a containerd configuration file at the location `/etc/containerd/config.toml`
    
    ```bash
    sudo mkdir -p /etc/containerd
    sudo containerd config default | sudo tee /etc/containerd/config.toml
    ```
    
    Now, we enable the systemd cgroup driver for the CRI in `/etc/containerd/config.toml` at section `[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]` set `SystemCgroup = true`
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718901406563/230cf196-74be-4fd7-b7df-c12b66a6c411.png align="center")
    
    OR we can just run
    
    ```bash
    sudo sed -i "s/SystemdCgroup = false/SystemdCgroup = true/g" "/etc/containerd/config.toml"
    ```
    
    Restart containerd
    
    ```bash
    sudo systemctl restart containerd
    ```
    
    Containerd should be running, check the status using:
    
    ```bash
    sudo systemctl status containerd
    ```
    

## Install kubeadm, kubelet and kubectl

***Run these commands on all nodes. These instructions are for Kubernetes v1.30.***

* **Install**`apt-transport-https, ca-certificates, curl, gpg`**packages**
    
    ```bash
    sudo apt-get update
    sudo apt-get install -y apt-transport-https ca-certificates curl gpg
    ```
    
* **Download the public signing key for the Kubernetes package repositories**
    
    ```bash
    curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
    ```
    
* **Add the**`apt`**repository for Kubernetes 1.30**
    
    ```bash
    echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
    ```
    
* **Install kubelet, kubeadm and kubectl**
    
    ```bash
    sudo apt-get update
    sudo apt-get install -y kubelet kubeadm kubectl
    sudo apt-mark hold kubelet kubeadm kubectl
    ```
    
* **This is optional, Enable the kubelet service before running kubeadm**
    
    ```bash
    sudo systemctl enable --now kubelet
    ```
    

## Initialize the k8s control plane

***Run these instructions only on the control plane node***

* ***kubeadm init***
    
    To initialize the control plane, run the `kubeadm init` command. You also need to choose a pod network add-on and deploy a `Container Network Interface (CNI)` so that your Pods can communicate with each other. Cluster DNS (CoreDNS) will not start until a network is installed.
    
    We will use Calico CNI, so set the `--pod-network-cidr=192.168.0.0/16`.
    
    ```bash
    sudo kubeadm init --pod-network-cidr=192.168.0.0/16
    ```
    
    Now, at the end of the `kubeadm init` command, you'll see `kubeadm join` command
    
    `sudo kubeadm join <control-plane-ip>:<control-plane-port> --token <token> --discovery-token-ca-cert-hash <hash>` copy it and keep it safe.
    
* **Run the following commands to set kubeconfig to access the cluster using kubectl**
    
    ```bash
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    ```
    
    Now, check the node using
    
    ```bash
    $ kubectl get nodes
    NAME             STATUS     ROLES           AGE    VERSION
    ip-zzz-zz-z-zz   NotReady   control-plane   114s   v1.30.2
    ```
    
    The node is in `NotReady` state because `message: 'container runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized' reason: KubeletNotReady` . After setting up the CNI, the node should be in a Ready state.
    
* **Set Up a Pod Network**
    
    We must deploy a Container Network Interface (CNI) based Pod network add-on so that Pods can communicate with each other.
    
    ***Install the calico operator on the cluster.***
    
    ```bash
    kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/tigera-operator.yaml
    ```
    
    ***Download the custom resources necessary to configure Calico.***
    
    ```bash
    kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/custom-resources.yaml
    ```
    
    ***Verify all the Calico pods are in running state.***
    
    ```bash
    $ kubectl get pods -n calico-system
    NAME                                       READY   STATUS    RESTARTS   AGE
    calico-kube-controllers-6f459db86d-mg657   1/1     Running   0          3m36s
    calico-node-ctc9q                          1/1     Running   0          3m36s
    calico-typha-774d5fbdb7-s7qsg              1/1     Running   0          3m36s
    csi-node-driver-bblm8                      2/2     Running   0          3m3
    ```
    
    ***Verify that the node is in a running state***
    
    ```bash
    $ kubectl get nodes
    NAME             STATUS   ROLES           AGE     VERSION
    ip-xxx-xx-x-xx   Ready    control-plane   2m46s   v1.30.2
    ```
    

## Join the worker nodes

Ensure `containerd`, `kubeadm`, `kubectl`, and `kubelet` are installed on all worker nodes, then run `sudo kubeadm join <control-plane-ip>:<control-plane-port> --token <token> --discovery-token-ca-cert-hash <hash>`, which you can find at the end of the `kubeadm init` command's output.

### Check the cluster state

***Run these commands on the control-plane node since the worker nodes do not have the kubeconfig file.***

* **Check if the worker nodes are joined to the cluster.**
    
    ```bash
    $ kubectl get nodes
    NAME               STATUS   ROLES           AGE    VERSION
    ip-xxx-xx-xx-xx    Ready    <none>          9m9s   v1.30.2
    ip-yyy-yy-yy-yy    Ready    <none>          23s    v1.30.2
    ip-zzz-zz-z-zz     Ready    control-plane   27m    v1.30.2
    ```
    
    To add a worker role to the worker node we can use `kubectl label node <node-name> node-role.kubernetes.io/worker=worker` command.
    
    ```bash
    $ kubectl get nodes
    NAME               STATUS   ROLES           AGE     VERSION
    ip-xxx-xx-xx-xx    Ready    worker          12m     v1.30.2
    ip-yyy-yy-yy-yy    Ready    worker          3m51s   v1.30.2
    ip-zzz-zz-z-zz     Ready    control-plane   31m     v1.30.2
    ```
    
* **Check the workload running on the cluster**
    
    ```bash
    $ kubectl get pods -A
    NAMESPACE          NAME                                       READY   STATUS    RESTARTS   AGE
    calico-apiserver   calico-apiserver-6fcb65fbd5-n4wsn          1/1     Running   0          35m
    calico-apiserver   calico-apiserver-6fcb65fbd5-nnggl          1/1     Running   0          35m
    calico-system      calico-kube-controllers-6f459db86d-mg657   1/1     Running   0          35m
    calico-system      calico-node-ctc9q                          1/1     Running   0          35m
    calico-system      calico-node-dmgt2                          1/1     Running   0          18m
    calico-system      calico-node-nw4t5                          1/1     Running   0          9m49s
    calico-system      calico-typha-774d5fbdb7-s7qsg              1/1     Running   0          35m
    calico-system      calico-typha-774d5fbdb7-sxb5c              1/1     Running   0          9m39s
    calico-system      csi-node-driver-bblm8                      2/2     Running   0          35m
    calico-system      csi-node-driver-jk4sz                      2/2     Running   0          18m
    calico-system      csi-node-driver-tbrrj                      2/2     Running   0          9m49s
    kube-system        coredns-7db6d8ff4d-5f7s5                   1/1     Running   0          37m
    kube-system        coredns-7db6d8ff4d-qj9r8                   1/1     Running   0          37m
    kube-system        etcd-ip-zzz-zz-z-zz                        1/1     Running   0          37m
    kube-system        kube-apiserver-ip-zzz-zz-z-zz              1/1     Running   0          37m
    kube-system        kube-controller-manager-ip-zzz-zz-z-zz     1/1     Running   0          37m
    kube-system        kube-proxy-dq8k4                           1/1     Running   0          9m49s
    kube-system        kube-proxy-t2sw9                           1/1     Running   0          18m
    kube-system        kube-proxy-xd6nn                           1/1     Running   0          37m
    kube-system        kube-scheduler-ip-zzz-zz-z-zz              1/1     Running   0          37m
    tigera-operator    tigera-operator-76ff79f7fd-jj4kf           1/1     Running   0          35m
    ```
    

## Conclusion

Setting up a Kubernetes cluster with Kubeadm involves a clear and structured process. You can create a functional cluster by meeting all prerequisites, configuring the container runtime, and installing Kubernetes components. Using Calico for networking ensures seamless pod communication. With the control plane and worker nodes properly configured and joined, you can efficiently manage and deploy workloads across your Kubernetes cluster.

***Thank you for reading this blog; your interest is greatly appreciated, and I hope it helps you on your Kubernetes journey; in the next article, we'll explore running workloads in the Kubernetes cluster.***