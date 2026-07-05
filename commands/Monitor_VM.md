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
# Install Blackbox Exporter

Blackbox Exporter enables Prometheus to monitor the availability of external services, websites, APIs, and applications through HTTP, HTTPS, TCP, DNS, and ICMP probes.

## Download Blackbox Exporter

```bash
wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.28.0/blackbox_exporter-0.28.0.linux-amd64.tar.gz
```

## Extract the Archive

```bash
tar -xvf blackbox_exporter-0.28.0.linux-amd64.tar.gz
```

Remove the downloaded archive:

```bash
rm -rf blackbox_exporter-0.28.0.linux-amd64.tar.gz
```

Navigate to the extracted directory:

```bash
cd blackbox_exporter-0.28.0.linux-amd64
```

Verify the contents:

```bash
ls
```

## Start Blackbox Exporter

```bash
./blackbox_exporter &
```

> The `&` symbol runs the exporter in the background.

By default, Blackbox Exporter listens on port **9115**.

Access it through:

```text
http://<MONITOR_VM_IP>:9115
```

---

# Configure Prometheus

Return to the Prometheus directory:

```bash
cd ..
cd prometheus-3.10.0.linux-amd64
```

Edit the configuration file:

```bash
vi prometheus.yml
```

Add the following section inside `scrape_configs`:

```yaml
- job_name: 'blackbox'

  metrics_path: /probe

  params:
    module: [http_2xx]

  static_configs:
    - targets:
      - http://prometheus.io
      - http://54.89.4.235:31740

  relabel_configs:
    - source_labels: [__address__]
      target_label: __param_target

    - source_labels: [__param_target]
      target_label: instance

    - target_label: __address__
      replacement: 13.218.215.242:9115
```

### Explanation

* `http_2xx` checks whether the target responds with an HTTP 200 status code.
* `targets` specifies the endpoints to monitor.
* `replacement` indicates the address of the Blackbox Exporter instance.
* Prometheus sends probe requests to Blackbox Exporter, which then probes the target systems.

---

# Restart Prometheus

Find the running Prometheus process:

```bash
pgrep prometheus
```

Terminate the process:

```bash
kill <PID>
```

Restart Prometheus:

```bash
./prometheus &
```

---

## Notes

* **Prometheus** collects metrics.
* **Blackbox Exporter** performs active probing of endpoints.
* **Grafana** visualizes collected metrics.
* Blackbox Exporter runs by default on port **9115**.
* Prometheus runs by default on port **9090**.
* Grafana runs by default on port **3000**.
* Blackbox Exporter is useful for monitoring application availability and uptime.
## Update the Prometheus Configuration

After installing **Node Exporter** on the Jenkins VM and enabling the **Prometheus Metrics Plugin** in Jenkins, Prometheus must be configured to scrape these metrics.

Navigate to the Prometheus directory:

```bash
cd prometheus-3.10.0.linux-amd64
```

Edit the Prometheus configuration file:

```bash
vi prometheus.yml
```

Add the following jobs inside the `scrape_configs` section:

```yaml
- job_name: 'node_exporter'
  static_configs:
    - targets:
      - '<JENKINS_IP>:9100'

- job_name: 'jenkins'
  metrics_path: /prometheus
  static_configs:
    - targets:
      - '<JENKINS_IP>:8080'
```

### Restart Prometheus

Locate the running Prometheus process:

```bash
pgrep prometheus
```

Stop the process:

```bash
kill <PID>
```

Start Prometheus again:

```bash
./prometheus &
```

### Notes

* **Node Exporter** exposes infrastructure metrics on port **9100**.
* The **Prometheus Metrics Plugin** exposes Jenkins metrics at `/prometheus`.
* Prometheus periodically scrapes these endpoints and stores the collected metrics.
* Grafana can then be used to visualize Jenkins and system metrics through dashboards.

