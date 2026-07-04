# Master VM Setup

## Kubernetes Cluster Installation

This document describes the steps performed to prepare the master virtual machine and install Kubernetes components.

### 1. Create the installation script

```bash
vi install_K8s.sh
```

### 2. Update package repositories

```bash
sudo apt-get update
```

### 3. Install Docker

```bash
sudo apt install docker.io -y
```

Allow non-root access to the Docker socket:

```bash
sudo chmod 666 /var/run/docker.sock
```

### 4. Install required dependencies

```bash
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg
```

Create the Kubernetes keyring directory:

```bash
sudo mkdir -p -m 755 /etc/apt/keyrings
```

### 5. Add the Kubernetes repository

Download the Kubernetes signing key:

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key \
| sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

Add the Kubernetes package repository:

```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' \
| sudo tee /etc/apt/sources.list.d/kubernetes.list
```

### 6. Make the installation script executable

```bash
chmod +x install_K8s.sh
```

### 7. Execute the installation script

```bash
./install_K8s.sh
```
### 8. Install Kubernetes Components

> **Note:** Using the latest Kubernetes version may sometimes introduce compatibility issues or bugs. For a stable and reproducible environment, it is often preferable to install a specific version.

Update package repositories:

```bash
sudo apt update
```

Install Kubernetes components with a fixed version:

```bash
sudo apt install -y kubeadm=1.28.1-1.1 \
kubelet=1.28.1-1.1 \
kubectl=1.28.1-1.1
```

This ensures that all Kubernetes components use the same version (**v1.28.1**), avoiding potential issues caused by installing the latest available release.
### 9. Initialize the Kubernetes Cluster

```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16



## Configure kubectl Access (Master Node)

After initializing the cluster, configure `kubectl` for the current user.

Create the Kubernetes configuration directory:

```bash
mkdir -p $HOME/.kube
```

Copy the administrator configuration file:

```bash
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```

Grant ownership of the configuration file to the current user:

```bash
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

---

## Install the Calico CNI Plugin

Deploy Calico to enable networking between Pods in the cluster.

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

**Note:** A Container Network Interface (CNI) plugin is required because Kubernetes does not provide Pod networking by default.

---

## Install the NGINX Ingress Controller

Deploy the NGINX Ingress Controller for external access to services.

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.49.0/deploy/static/provider/baremetal/deploy.yaml
```

### Purpose

* **Calico** → Provides Pod-to-Pod networking across the cluster.
* **Ingress NGINX** → Manages incoming HTTP/HTTPS traffic and routes it to Kubernetes Services.
* **kubectl configuration** → Allows the current user to interact with the cluster without using `sudo`.

