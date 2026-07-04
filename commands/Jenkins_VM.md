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
