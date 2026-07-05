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


## Verify Cluster Status

Check that all nodes have successfully joined the cluster.

```bash
kubectl get nodes
```

This command displays the status of the control plane and worker nodes.

---

## Kubernetes Security Audit with kubeaudit

Although **Trivy** can scan Kubernetes resources, it may not always fit every environment or use case. As an alternative, **kubeaudit** can be used to audit the cluster against Kubernetes security best practices.

### Install kubeaudit

Download the release package:

```bash
wget https://github.com/Shopify/kubeaudit/releases/download/v0.22.2/kubeaudit_0.22.2_linux_amd64.tar.gz
```

Extract the archive:

```bash
tar -xvzf kubeaudit_0.22.2_linux_amd64.tar.gz
```

Move the binary to a directory included in the system PATH:

```bash
sudo mv kubeaudit /usr/local/bin/
```

Verify the installation:

```bash
kubeaudit version
```

Run a complete security audit:

```bash
kubeaudit all
```

### Notes

* `kubectl get nodes` verifies that the cluster is operational.
* `kubeaudit` checks Kubernetes resources for security misconfigurations.
* `kubeaudit all` runs all available audit checks.
* Trivy and kubeaudit are complementary tools and can both be used in Kubernetes security assessments.


# Jenkins Access Configuration for Kubernetes

To allow Jenkins to deploy applications into the Kubernetes cluster, a dedicated **ServiceAccount**, **Role**, **RoleBinding**, and **Secret** are created.

---

## 1. Create a Namespace

Create a dedicated namespace where applications will be deployed.

```bash
kubectl create ns webapps
```

---

## 2. Create a Service Account

A ServiceAccount represents a separate identity inside Kubernetes that Jenkins will use for deployments.

Create `svc.yaml`:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: webapps
```

Apply the manifest:

```bash
kubectl apply -f svc.yaml
```

### Notes

* `jenkins` is the ServiceAccount name.
* It will perform deployments inside the `webapps` namespace.
* This creates a dedicated Kubernetes user for Jenkins.

---

## 3. Create a Role

A Role defines the permissions granted within a namespace.

Create `role.yaml`:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role

metadata:
  name: jenkins-role
  namespace: webapps

rules:
- apiGroups:
  - ""
  - apps
  - autoscaling
  - batch
  - extensions
  - policy
  - rbac.authorization.k8s.io

  resources:
  - pods
  - secrets
  - configmaps
  - daemonsets
  - deployments
  - events
  - endpoints
  - horizontalpodautoscalers
  - ingresses
  - jobs
  - limitranges
  - namespaces
  - nodes
  - persistentvolumes
  - persistentvolumeclaims
  - resourcequotas
  - replicasets
  - replicationcontrollers
  - serviceaccounts
  - services

  verbs:
  - get
  - list
  - watch
  - create
  - update
  - patch
  - delete
```

Apply the Role:

```bash
kubectl apply -f role.yaml
```

### Notes

The role allows Jenkins to:

* Get resources
* List resources
* Watch resources
* Create resources
* Update resources
* Patch resources
* Delete resources

---

## 4. Create a RoleBinding

A RoleBinding associates the Role with the Jenkins ServiceAccount.

Create `bind.yaml`:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding

metadata:
  name: jenkins-rolebinding
  namespace: webapps

roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: jenkins-role

subjects:
- kind: ServiceAccount
  name: jenkins
  namespace: webapps
```

Apply the binding:

```bash
kubectl apply -f bind.yaml
```

### Notes

The RoleBinding grants the permissions defined in `jenkins-role` to the `jenkins` ServiceAccount.

This enables Jenkins to interact with the Kubernetes cluster and deploy applications inside the `webapps` namespace.

---

## 5. Create a Service Account Token

Create `secret.yaml`:

```yaml
apiVersion: v1
kind: Secret

type: kubernetes.io/service-account-token

metadata:
  name: mysecretname

  annotations:
    kubernetes.io/service-account.name: jenkins
```

Apply the secret:

```bash
kubectl apply -f secret.yaml -n webapps
```

Retrieve the generated token:

```bash
kubectl describe secret mysecretname -n webapps
```

### Notes

* The token will later be added to Jenkins as Kubernetes credentials.
* Jenkins uses this token to authenticate against the Kubernetes API Server.
* The ServiceAccount, Role, RoleBinding, and Secret together implement Kubernetes RBAC for Jenkins deployments.





