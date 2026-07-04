# Docker Installation (Required for Nexus)

This setup must be performed on the VM hosting SonarQube or Nexus, as both services will run inside Docker containers.

## Update Package Index

```bash
sudo apt update
```

## Create the Docker Installation Script

```bash
vi docker.sh
```

Paste the following commands into `docker.sh`.

### Add Docker's Official GPG Key

```bash
sudo apt update
sudo apt install -y ca-certificates curl

sudo install -m 0755 -d /etc/apt/keyrings

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
-o /etc/apt/keyrings/docker.asc

sudo chmod a+r /etc/apt/keyrings/docker.asc
```

### Add the Docker Repository

```bash
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```

### Refresh Packages

```bash
sudo apt update
```

### Install Docker Engine

```bash
sudo apt install -y docker-ce docker-ce-cli \
containerd.io docker-buildx-plugin \
docker-compose-plugin
```

## Make the Script Executable

```bash
sudo chmod +x docker.sh
```

## Execute the Script

```bash
./docker.sh
```

## Allow Non-Root Users to Run Docker Commands

By default, only the `root` user can execute Docker commands. To allow other users to interact with Docker, run:

```bash
sudo chmod 666 /var/run/docker.sock
```

## Verification

Check that Docker is installed correctly:

```bash
docker --version
```

Verify that Docker is running:

```bash
docker ps
```

---

### Notes

* **Nexus Repository Manager** will also run inside a Docker container.
* Using Docker simplifies deployment, upgrades, and maintenance.
