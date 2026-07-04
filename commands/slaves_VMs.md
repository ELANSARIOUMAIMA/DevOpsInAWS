# Slave 1 & 2 VMs Setup

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


