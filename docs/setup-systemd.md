# Setting Dashboard as Systemd Service

```bash
REPODIR=/home/user/repos
```

## Setup
```bash
cd $REPODIR
git clone https://github.com/elvinongbl/e2e-observability.git
cd e2e-observability
```

## Node Exporter
Note: Adjust below version accordingly if you need new version:
```bash
cd 3rd-party
wget https://github.com/prometheus/node_exporter/releases/download/v1.10.2/node_exporter-1.10.2.linux-amd64.tar.gz
tar -xvf node_exporter-1.10.2.linux-amd64.tar.gz

# Run node-exporter on a new terminal
cd node_exporter-1.10.2.linux-amd64

# Copy node_exporter to /usr/bin
sudo cp node_exporter /usr/bin

# Copy systemd/e2e_node_exporter.service to systemd
cd $REPODIR/e2e-observability
sudo cp systemd/e2e_node_exporter.service /etc/systemd/system

# Reload and start node_exporter.service
sudo systemctl daemon-reload
sudo systemctl enable --now e2e_node_exporter
sudo systemctl status e2e_node_exporter

# Test node-exporter on another terminal
curl http://localhost:9100/metrics
```

## Qmassa

### Rust setup
You may use the default installation path that install Rust and Cargo under $HOME:-
- Cargo --> /home/user/.cargo

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

### Install qmassa and qmmd

Project https://github.com/ulissesf/qmassa.git has qmassa (TUI) an qmmd (metrics exporter) for Prometheus.

```bash
# Install from cargo
# Below qmmd and qmassa are installed under $HOME/.cargo/bin/
cargo install --locked qmmd qmassa

# Make qmmd and qmassa available to system / sudo
sudo ln -sf /home/user/.cargo/bin/qmassa /usr/bin/qmassa
sudo ln -sf /home/user/.cargo/bin/qmmd /usr/bin/qmmd

# Look-up PCI-ID for intel iGPU and dGPU using lspci
lspci | grep VGA
# Example of
---
00:02.0 VGA compatible controller: Intel Corporation YYYY [Intel Graphics]
05:00.0 VGA compatible controller: Intel Corporation Device XXXX
---

# Make copy of systemd service according to your platform
sudo cp systemd/e2e_qmmd_exporter.service /etc/systemd/system/e2e_qmmd_exporter_igpu.service
sudo cp systemd/e2e_qmmd_exporter.service /etc/systemd/system/e2e_qmmd_exporter_dgpu.service

# Note:# In order to obtain all GPU metrics, User=root is used.
# Edit iGPU and dGPU service according to the port and PCI ID
# Example:
# Run qmmd for iGPU on a new terminal
# Port 9200 for iGPU, please change accordingly.
sudo nano /etc/systemd/system/e2e_qmmd_exporter_igpu.service
---
ExecStart=qmmd -d 0000:00:02.0 -p 9200
SyslogIdentifier=e2e_qmmd_exporter_igpu
---

# Run qmmd for dGPU on a new terminal
# Port 9300 for dGPU, please change accordingly.
sudo nano /etc/systemd/system/e2e_qmmd_exporter_dgpu.service
---
ExecStart=qmmd -d 0000:05:00.0 -p 9300
SyslogIdentifier=e2e_qmmd_exporter_dgpu
---

# Reload and start node_exporter.service
sudo systemctl daemon-reload
# For iGPU
sudo systemctl enable --now e2e_qmmd_exporter_igpu
sudo systemctl status e2e_qmmd_exporter_igpu

# For dGPU
sudo systemctl enable --now e2e_qmmd_exporter_dgpu
sudo systemctl status e2e_qmmd_exporter_dgpu

# Confirm node exporter for iGPU and dGPU are running (on another terminal)
curl http://localhost:9200/metrics
curl http://localhost:9300/metrics
```

## NPU Monitoring Tool
Project https://github.com/open-edge-platform/edge-ai-libraries/tree/main/tools/npu-monitor-tool contain
TUI and Prometheus-based metrics exporter for NPU monitoring.

Note: Prometheus enabling is now part of Open Edge Platform's edge-ai-libraries main branch at
[PR merged](https://github.com/open-edge-platform/edge-ai-libraries/commit/807ef9b500f0a82b5a51744c480138433fd959d6)
 
Note: The Prometheus metrics exporter depends on Python package and the sample systemd at systemd/e2e_npu_exporter.service
      is using pyenv that is installed for root. To setup pyenv for your system, please follow [docs/pyenv-setup.md](./docs/pyenv-setup.md).

```bash
# Assuming the PR has been merged
cd 3rd-party
git clone https://github.com/open-edge-platform/edge-ai-libraries
cd edge-ai-libraries/tools/npu-monitor-tool
# Change to root
sudo su
pip install -r requirements.txt
# Change back to user
su user

sudo cp systemd/e2e_npu_exporter.service /etc/systemd/system/e2e_npu_exporter.service
# Modify path to npu-monitor-tool
# Modify path to gunicorn under 'root' accordingly if you are not using pyenv
# Port=9400 assumed.
---
User=root
WorkingDirectory=/home/user/repos/e2e-observability/3rd-party/edge-ai-libraries/tools/npu-monitor-tool

ExecStart=/root/.pyenv/shims/gunicorn -w 1 -b localhost:9400 npu-metrics-exporter:app
SyslogIdentifier=e2e_npu_exporter
---

sudo systemctl daemon-reload
sudo systemctl enable --now e2e_npu_exporter
sudo systemctl restart e2e_npu_exporter
sudo systemctl status e2e_npu_exporter

# Confirm node exporter for NPU (on another terminal)
curl http://localhost:9400/metrics
```

## Nvidia GPU Exporter (community version)

```bash
cd 3rd-party
wget https://github.com/utkuozdemir/nvidia_gpu_exporter/releases/download/v1.4.1/nvidia-gpu-exporter_1.4.1_linux_amd64.deb
sudo dpkg -i nvidia-gpu-exporter_1.4.1_linux_amd64.deb
rm nvidia-gpu-exporter_1.4.1_linux_amd64.deb

# Create nvidia_gpu_exporter user
sudo useradd --system --no-create-home --shell /usr/sbin/nologin nvidia_gpu_exporter
sudo cp /usr/lib/systemd/system/nvidia_gpu_exporter.service /etc/systemd/system

# Modify Port=9500
sudo nano /etc/systemd/system/nvidia_gpu_exporter.service
---
ExecStart=/usr/bin/nvidia_gpu_exporter --web.listen-address=:9500
---

sudo systemctl daemon-reload
sudo systemctl enable --now nvidia_gpu_exporter
sudo systemctl status nvidia_gpu_exporter

# Check NVIDIA GPU metrics
curl http://localhost:9500/metrics
```

## Prometheus

Below prometheus yaml config is at [prometheus.yml](./prometheus.yml)

```bash
cd 3rd-party
wget https://github.com/prometheus/prometheus/releases/download/v3.8.1/prometheus-3.8.1.linux-amd64.tar.gz
tar -xvf prometheus-3.8.1.linux-amd64.tar.gz
# The package contains prometheus and sample prometheus.yml to run prometheus service

cd prometheus-3.8.1.linux-amd64
sudo cp prometheus /usr/bin

# You may copy or edit-then-copy the e2e-observability/prometheus.yml to /etc.
cd $REPODIR/e2e-observability
sudo cp prometheus.yml /etc/e2e-prometheus.yml
nano prometheus.yml
---
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # Node Exporter
  - job_name: "node exporter"
    scheme: http
    static_configs:
      - targets: ['localhost:9100']

  - job_name: "qmmd exporter iGPU"
    scheme: http
    static_configs:
      - targets: ['localhost:9200']

  - job_name: "qmmd exporter dGPU"
    scheme: http
    static_configs:
      - targets: ['localhost:9300']

  - job_name: "NPU Metrics Exporter"
    scheme: http
    static_configs:
      - targets: ['localhost:9400']

  - job_name: "NVDIA GPU Exporter"
    scheme: http
    static_configs:
      - targets: ['localhost:9500']
---

# Copy systemd/e2e_node_exporter.service to systemd
sudo cp systemd/e2e_prometheus.service /etc/systemd/system

# Reload and start node_exporter.service
sudo systemctl daemon-reload
sudo systemctl enable --now e2e_prometheus
sudo systemctl status e2e_prometheus

```

To check Promethues, use web-browser http://localhost:9090/query and look for metrics
with prefix, e.g node_, qmmd_, npu_

## Grafana
Assume your on Ubuntu 24.04:
```bash
# https://grafana.com/docs/grafana/latest/setup-grafana/installation/debian/
$ sudo mkdir -p /etc/apt/keyrings/
$ wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null

$ echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
$ sudo apt-get update
$ sudo apt-get install grafana

# Add proxy setting in /etc/default/grafana-server
$ sudo cat << EOF > /etc/default/grafana-server
HTTP_PROXY=http://<PROXY_SERVER>:<PORT>
HTTPS_PROXY=http://<PROXY_SERVER>:<PORT>
NO_PROXY=localhost,127.0.0.1
EOF

# Enable and start Grafana server
$ sudo systemctl enable grafana-server
$ sudo systemctl restart grafana-server
$ sudo systemctl status grafana-server
```

## Open Grafana Webpage
- Connect to Grafana Server using http://localhost:3000
- Use initial login/password  = admin/admin
- Update Grafana server with new password

## Create Data-source for Prometheus
- From Grafana left navigation menu, select 'Connections/Data Sources'.
  Under "Add data source", search for "Prometheus" and select it.
- In the "Prometheus/Settings" tab:
  - Name = Prometheus Data Feeder
  - Prometheus server URL = http://localhost:9090
- Then, scroll to the bottom, click 'Save & test'.
- Confirm that above "Prometheus Data Feeder" is listed (created) under Data sources,
  when you select Grafana left navigation 'Connections/Data Sources' menu again.

## Create New Dashboard for E2E Observability Dashboard
In Grafana page, create new dashboard and import the sample E2E metrics dashboard in
[grafana-e2e-dashboard.json](./grafana-e2e-dashboard.json)
