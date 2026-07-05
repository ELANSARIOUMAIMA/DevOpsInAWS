# Monitoring VM Setup

The monitoring node is responsible for collecting metrics and visualizing them using **Prometheus** and **Grafana**.

---

# Install Prometheus

Update package repositories:

```bash
sudo apt update
```

Download Prometheus:

```bash
wget https://github.com/prometheus/prometheus/releases/download/v3.10.0/prometheus-3.10.0.linux-amd64.tar.gz
```

Extract the archive:

```bash
tar -xvf prometheus-3.10.0.linux-amd64.tar.gz
```

Remove the downloaded archive:

```bash
rm -rf prometheus-3.10.0.linux-amd64.tar.gz
```

Navigate to the extracted directory:

```bash
cd prometheus-3.10.0.linux-amd64
```

List the contents:

```bash
ls
```

Start Prometheus:

```bash
./prometheus &
```

> The symbol `&` runs the process in the background, allowing the terminal to remain available for other commands.

Prometheus is accessible at:

```text
http://<MONITOR_VM_IP>:9090
```

---

# Install Grafana

Return to the previous directory:

```bash
cd ..
```

Install required dependencies:

```bash
sudo apt-get install -y adduser libfontconfig1 musl
```

Download Grafana Enterprise:

```bash
wget https://dl.grafana.com/grafana-enterprise/release/12.4.1/grafana-enterprise_12.4.1_22846628243_linux_amd64.deb
```

Install Grafana:

```bash
sudo dpkg -i grafana-enterprise_12.4.1_22846628243_linux_amd64.deb
```

Start the Grafana service:

```bash
sudo systemctl start grafana-server
```

Verify the service status:

```bash
sudo systemctl status grafana-server
```

Grafana is accessible at:

```text
http://<MONITOR_VM_IP>:3000
```

Default credentials:

```text
Username: admin
Password: admin
```

---

## Notes

* **Prometheus** collects and stores metrics from infrastructure and applications.
* **Grafana** provides dashboards for visualizing Prometheus metrics.
* Prometheus listens on port **9090** by default.
* Grafana listens on port **3000** by default.
* Running Prometheus with `&` keeps it executing in the background.
* Opening ports in the range **3000–10000** is useful because many DevOps tools expose web interfaces within this interval.
