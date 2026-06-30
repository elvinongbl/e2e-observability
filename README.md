# Overview

This is personal end-to-end observability dashboard that pull together below open-source projects
to allow me to monitor Intel CPU, GPU and NPU behaviour during day-to-day work.

Disclaimer: there is no warranty in adopting this dashboard.

Being a dashboard built on-top of Grafana/Prometheus, you may modify the feel and look according
to your preference.

[2026/04/30] Added Nvida Metrics Dashboard into E2E Dashboard for ease of comparison.
[2026/06/30] Added Dockerized recipe

# Example Grafana Dashboard

![E2E Observability](docs/images/e2e-cpu-gpu-npu.png)

# Software Architecture

![SW Arch](docs/images/e2e-sw-arch.png)

# Setup

Two deployment options are provided:
- A) **Docker Compose** (recommended for quick start) – see [Docker Setup](#docker-setup) below.
- B) **Bare-metal / systemd** – follow the individual sections further down.

## A) Docker Setup (Recommended)

[docker/dashboard-docker-compose.yml](docker/dashboard-docker-compose.yml) is used to start Grafana/Prometheus-based
dashboarding that provides the following:
- Intel CPU metrics (for example, CPU, memory, disk, and network utilization) via node-exporter
- Intel GPU metrics (iGPU or dGPU) via qmassa/qmmd
- Intel NPU metrics via Open Edge Platform / Edge AI Libraries / tools / npu-monitoring-tool
- NVIDIA GPU metrics via nvidia_gpu_exporter

### NVIDIA GPU Container Tookit
Please follow [docs/setup-nv-gpu.md](docs/setup-nv-gpu.md) for installing NVIDIA Container Toolkit.

## Start/Stop Dashboarding Docker

### [Optional] Disable Systemd Services

If you added the systemd approach (B), please disable all systemd services before starting the Dockerized Dashboard method.

```bash
# To disable systemd services
sudo systemctl disable ese_node_exporter.service
sudo systemctl stop ese_node_exporter.service

sudo systemctl disable ese_npu_exporter.service
sudo systemctl stop ese_npu_exporter.service

sudo systemctl disable e2e_qmmd_exporter_dgpu.service
sudo systemctl stop e2e_qmmd_exporter_dgpu.service

sudo systemctl disable e2e_qmmd_exporter_igpu.service
sudo systemctl stop e2e_qmmd_exporter_igpu.service

sudo systemctl disable e2e_prometheus.service
sudo systemctl stop e2e_prometheus.service

sudo systemctl disable grafana-server
sudo systemctl stop grafana-server
```

### Start/Stop Dockerized Dashboard

[docker/.env.sample](docker/.env.sample) contain the sample environment for the Docker containter setup.

Note: If you plan to change the port number, please make sure matching port in [docker/prometheus/prometheus.yml](docker/prometheus/prometheus.yml).

```bash
# Note: Remove '--profile nvidia' if NVIDIA GPU is not required

# Make a copy of environment and adjust the settings (optional).
cp docker/.env.sample docker/.env

# Start Dashboard. 
sudo docker compose -f docker/dashboard-docker-compose.yml up --profile nvidia -d

# Stop Dashboard
sudo docker compose -f docker/dashboard-docker-compose.yml --profile nvidia down
```

## B) Bare-metal / systemd Setup (Optional)

Refer to [docs/setup-systemd.md](docs/setup-systemd.md)

## Debugging
```bash
# Use before commands to check which metrics exporter is broken

# CPU Node Exporter
curl http://localhost:9100/metrics

# iGPU
curl http://localhost:9200/metrics

# dGPU
curl http://localhost:9300/metrics

# NPU
curl http://localhost:9400/metrics

# NVIDIA GPU
curl http://localhost:9500/metrics

# Prometheus
# webpage=http://localhost:9090/query

# Grafana
# webpage=http://localhost:3000
```
