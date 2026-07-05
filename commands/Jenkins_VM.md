# Jenkins VM Setup

## Prerequisites

Update package repositories:

```bash
sudo apt update
```

Jenkins requires **Java 17 or later** to run.

The `-y` option automatically answers **"yes"** to installation prompts, allowing the installation to proceed without manual confirmation.

Install Java:

```bash
sudo apt install openjdk-17-jre-headless -y
```

Verify the installation:

```bash
java -version
```

---

## Install Jenkins

The installation steps can be found in the official Jenkins documentation by selecting:

**Install Jenkins → Linux**

Create an installation script:

```bash
vi jenkins.sh
```

Make the script executable:

```bash
sudo chmod +x jenkins.sh
```

Execute the script:

```bash
./jenkins.sh
```

### Add the Jenkins Repository

Download the Jenkins signing key:

```bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
```

Add the Jenkins repository:

```bash
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null
```

Update package information:

```bash
sudo apt update
```

Install Jenkins:

```bash
sudo apt install jenkins -y
```

---

## Install Docker

Jenkins pipelines frequently execute Docker commands, so Docker must also be installed on the Jenkins VM.

You can reuse the same `docker.sh` script created for the SonarQube VM.

Display the existing script:

```bash
cat docker.sh
```

Create a new script on the Jenkins VM:

```bash
vi docker.sh
```

Paste the Docker installation commands, then execute:

```bash
sudo chmod +x docker.sh
./docker.sh
```

Alternatively, install Docker directly:

```bash
sudo apt install docker-ce docker-ce-cli \
containerd.io docker-buildx-plugin \
docker-compose-plugin -y
```

Allow users other than root to execute Docker commands:

```bash
sudo chmod 666 /var/run/docker.sock
```

---

## Retrieve the Initial Jenkins Password

Obtain the administrator password:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

---

## Access Jenkins

Open your browser and navigate to:

```text
http://<VM_IP>:8080
```

Paste the retrieved password to complete the initial configuration.

---

## Notes

* Jenkins requires **Java 17+**.
* The `-y` option automatically confirms installation prompts.
* Docker is installed because Jenkins pipelines often build images, run containers, and interact with registries.
* The default Jenkins port is **8080**.

## Jenkins Configuration

After installing Jenkins, several plugins must be added to support the CI/CD pipeline.

### Required Plugins

Install the following plugins from:

**Manage Jenkins → Plugins**

| Plugin                     | Purpose                                                                      |
| -------------------------- | ---------------------------------------------------------------------------- |
| Eclipse Temurin Installer  | Automatically installs and manages multiple JDK versions                     |
| Config File Provider       | Allows creation and management of configuration files such as `settings.xml` |
| Pipeline Maven Integration | Integrates Maven builds with Jenkins Pipelines                               |
| SonarQube Scanner          | Enables code analysis with SonarQube                                         |
| Docker                     | Adds Docker support within Jenkins                                           |
| Docker Pipeline            | Allows Docker commands inside Jenkins pipelines                              |
| Kubernetes                 | Integrates Jenkins with Kubernetes                                           |
| Kubernetes CLI             | Provides access to `kubectl` commands in pipelines                           |
| Kubernetes Client API      | Enables communication with Kubernetes clusters                               |
| Kubernetes Credentials     | Manages Kubernetes authentication credentials                                |
| Prometheus Metric    | Exposes Jenkins metrics in a format that Prometheus can scrape                                |

---

## Configure Global Tools

Navigate to:

```text
Manage Jenkins
    └── Tools
```

Configure the following tools:

### JDK Configuration

```text
Name: jdk17
```

Install automatically using the **Eclipse Temurin Installer** plugin.

### Maven Configuration

```text
Name: maven3
```

Install automatically and select the desired Maven version.

### SonarQube Scanner Configuration

```text
Name: sonar-scanner
```

Install automatically and select the latest available version.

### Docker Configuration

```text
Name: docker
```

Configure the Docker installation to allow Jenkins pipelines to execute Docker commands.

---

## Notes

* **jdk17** → Java runtime required by Jenkins and Java applications.
* **maven3** → Build automation and dependency management tool.
* **sonar-scanner** → Performs static code analysis and publishes results to SonarQube.
* **docker** → Containerization platform used to build, run, and manage containers.
* **Eclipse Temurin Installer** → Simplifies JDK installation and management.
* **Config File Provider** → Used to create and manage files such as `settings.xml`.
* **Pipeline Maven Integration** → Enables Maven builds within Jenkins Pipelines.
* **SonarQube Scanner** → Allows Jenkins to trigger code quality analysis.
* **Docker** and **Docker Pipeline** → Enable image creation and container management inside Jenkins Pipelines.
* **Kubernetes**, **Kubernetes CLI**, **Kubernetes Client API**, and **Kubernetes Credentials** → Enable Jenkins integration with Kubernetes clusters.

## Install Trivy on the Jenkins VM

Trivy is used to scan source code, container images, filesystems, and Kubernetes resources for vulnerabilities and misconfigurations.

Create an installation script:

```bash
vi trivy.sh
```

Add the following commands to the script.

### Install Required Dependencies

```bash
sudo apt-get install wget apt-transport-https gnupg lsb-release
```

### Add the Trivy Repository Key

```bash
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key \
| gpg --dearmor \
| sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
```

### Add the Trivy Repository

```bash
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] \
https://aquasecurity.github.io/trivy-repo/deb generic main" \
| sudo tee -a /etc/apt/sources.list.d/trivy.list
```

### Install Trivy

```bash
sudo apt-get update
sudo apt-get install trivy -y
```

### Verify the Installation

```bash
trivy --version
```

---

## Notes

* **Trivy** is a security scanner developed by Aqua Security.
* It can scan container images, source code repositories, Kubernetes resources, and filesystems.
* In Jenkins pipelines, Trivy is commonly used to perform vulnerability assessments before deploying applications.
* The `-y` option automatically confirms installation prompts.
## Install kubectl on the Jenkins VM

Jenkins pipelines may execute Kubernetes commands such as `kubectl apply`, `kubectl get pods`, or `kubectl rollout status`. Therefore, `kubectl` must be installed on the Jenkins server.

Create an installation script:

```bash
vi kubectl.sh
```

Add the following commands:

```bash
curl -o kubectl \
https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl

chmod +x ./kubectl

sudo mv ./kubectl /usr/local/bin

kubectl version --short --client
```

Make the script executable:

```bash
sudo chmod +x kubectl.sh
```

Run the script:

```bash
./kubectl.sh
```

### Verification

```bash
kubectl version --short --client
```

### Notes

* **kubectl** is the command-line tool used to interact with Kubernetes clusters.
* Jenkins uses `kubectl` in pipelines to deploy and manage Kubernetes resources.
* Installing `kubectl` on the Jenkins VM allows Jenkins jobs to communicate with the Kubernetes API Server.
* Ensure that the `kubectl` version is compatible with the Kubernetes cluster version.
## Install Node Exporter

Node Exporter exposes system-level metrics such as CPU usage, memory consumption, disk utilization, and network statistics.

Download Node Exporter:

```bash
wget https://github.com/prometheus/node_exporter/releases/latest/download/node_exporter-*.linux-amd64.tar.gz
```

Extract the archive:

```bash
tar -xvf node_exporter-*.linux-amd64.tar.gz
```

Navigate to the extracted directory:

```bash
cd node_exporter-*.linux-amd64
```

Start Node Exporter:

```bash
./node_exporter &
```

Verify that Node Exporter is running:

```bash
curl localhost:9100/metrics
```

Node Exporter exposes metrics on port **9100** by default.

Access the metrics endpoint:

```text
http://<JENKINS_IP>:9100
```

### Notes

* **Node Exporter** collects operating system metrics from the Jenkins VM.
* Prometheus scrapes these metrics periodically.
* The default port for Node Exporter is **9100**.
* Grafana dashboards can be built using the collected Node Exporter metrics.


