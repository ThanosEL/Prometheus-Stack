# Prometheus Stack Setup Guide

Setup guide for Prometheus, Node Exporter, Alertmanager, Grafana on Ubuntu, and Windows Exporter on Windows.

---

## Table of Contents

- [Prometheus](#1-prometheus)
- [Node Exporter](#2-node-exporter)
- [Alertmanager](#3-alertmanager)
- [Grafana](#4-grafana)
- [Windows Exporter](#5-windows-exporter)

---

## 1. Prometheus

### Create User and Group

```bash
sudo groupadd --system prometheus
sudo useradd -s /sbin/nologin --system -g prometheus prometheus
```

### Create Required Directories

```bash
sudo mkdir /var/lib/prometheus
sudo mkdir -p /etc/prometheus/rules
sudo mkdir -p /etc/prometheus/rules.s
sudo mkdir -p /etc/prometheus/files_sd
```

### Download and Extract

```bash
sudo wget <link-to-prometheus-package>
sudo tar xvf prometheus.tar.gz
```

### Move Binaries and Config

```bash
sudo mv prometheus promtool /usr/local/bin/
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
```

### Create Systemd Service

```bash
sudo tee /etc/systemd/system/prometheus.service << EOF
<content-from-prometheus.service-file>
EOF
```

> Reference: [prometheus.service](https://github.com/aussiearef/Prometheus/blob/main/prometheus.service)

### Set Ownership and Permissions

```bash
sudo chown -R prometheus:prometheus /etc/prometheus
sudo chown -R prometheus:prometheus /var/lib/prometheus
sudo chmod -R 755 /etc/prometheus
sudo chmod -R 755 /etc/prometheus/*
```

### Start Service

```bash
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus
sudo systemctl status prometheus
```

### Web UI

```
http://<your_server_ip>:9090
```

---

## 2. Node Exporter

### Download and Extract

```bash
sudo wget https://github.com/prometheus/node_exporter/releases/download/v1.9.1/node_exporter-1.9.1.linux-amd64.tar.gz
tar -xvf node_exporter-1.9.1.linux-amd64.tar.gz
```

### Create User and Group (if not already done)

```bash
sudo groupadd --system prometheus
sudo useradd -s /sbin/nologin --system -g prometheus prometheus
```

### Prepare Directory and Move Binary

```bash
sudo mkdir /var/lib/node/
sudo mv node_exporter /var/lib/node/
```

### Create Systemd Service

```bash
sudo tee /etc/systemd/system/node.service << EOF
<content-from-node.service-file>
EOF
```

> Reference: [node.service](https://github.com/aussiearef/Prometheus/blob/main/node.service)

### Set Ownership and Permissions

```bash
sudo chown -R prometheus:prometheus /var/lib/node/
sudo chmod -R 755 /var/lib/node/
```

### Start Service

```bash
sudo systemctl daemon-reload
sudo systemctl start node
sudo systemctl enable node
sudo systemctl status node
```

### Configure Prometheus to Scrape Node Exporter

Edit `/etc/prometheus/prometheus.yml` and add your Node Exporter target:

```yaml
scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['<node_exporter_host>:9100']
```

Then restart Prometheus:

```bash
sudo systemctl restart prometheus
```

### Metrics Endpoint

```
http://<node_exporter_host>:9100/metrics
```

---

## 3. Alertmanager

### Create User and Group (if not already done)

```bash
sudo groupadd --system prometheus
sudo useradd -s /sbin/nologin --system -g prometheus prometheus
```

### Create Directory and Move Binaries

```bash
sudo mkdir /var/lib/alert_manager/
sudo mv alertmanager amtool /var/lib/alert_manager/
```

### Create Systemd Service

```bash
sudo tee /etc/systemd/system/alert_manager.service << EOF
<content-from-alertmanager.service-file>
EOF
```

> Reference: [alertmanager.service](https://github.com/aussiearef/Prometheus/blob/main/alertmanager.service)

### Set Ownership and Permissions

```bash
sudo chown -R prometheus:prometheus /var/lib/alert_manager/
sudo chmod -R 755 /var/lib/alert_manager/
```

### Start Service

```bash
sudo systemctl daemon-reload
sudo systemctl start alert_manager
sudo systemctl enable alert_manager
sudo systemctl status alert_manager
```

### Web UI

```
http://localhost:9093
```

---

## 4. Grafana

### Install Dependencies

```bash
sudo apt-get install -y adduser libfontconfig1 musl
```

### Download and Install

```bash
wget https://dl.grafana.com/grafana-enterprise/release/12.2.0/grafana-enterprise_12.2.0_17949786146_linux_amd64.deb
sudo dpkg -i grafana-enterprise_12.2.0_17949786146_linux_amd64.deb
```

### Start Service

```bash
sudo systemctl daemon-reload
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```

### Web UI

```
http://localhost:3000
```

> Default credentials: `admin` / `admin` (change on first login)

---

## 5. Windows Exporter

### Download .msi

```powershell
Invoke-WebRequest -Uri "https://github.com/prometheus-community/windows_exporter/releases/download/v0.31.8/windows_exporter-0.31.8-amd64.msi" `
  -OutFile "C:\Users\Administrator\windows_exporter-0.31.8-amd64.msi"
```

### Verify Hash

```powershell
Get-FileHash C:\Users\Administrator\windows_exporter-0.31.8-amd64.msi -Algorithm SHA256
```

### Install

```powershell
msiexec /i C:\Users\Administrator\windows_exporter-0.31.8-amd64.msi /quiet `
  ENABLED_COLLECTORS="cpu,cs,logical_disk,memory,net,os,service,system,process,tcp"
```

### Check Service Status

```powershell
Get-Service windows_exporter
```

### Configure Service ImagePath

```powershell
$regPath = "HKLM:\SYSTEM\CurrentControlSet\Services\windows_exporter"
Set-ItemProperty -Path $regPath -Name "ImagePath" -Value `
  "`"C:\Program Files\windows_exporter\windows_exporter.exe`" --config.file=`"C:\Program Files\windows_exporter\config.yaml`""
```

### Start Service

```powershell
Start-Service windows_exporter
Get-Service windows_exporter
```

### Firewall Rule

```powershell
New-NetFirewallRule -DisplayName "Windows Exporter 9182" `
  -Direction Inbound -Protocol TCP -LocalPort 9182 -Action Allow
```

### Verify Firewall Rule

```powershell
Get-NetFirewallRule -DisplayName "*9182*" -ErrorAction SilentlyContinue
```

### Configure Prometheus to Scrape Windows Exporter

Edit `/etc/prometheus/prometheus.yml` and add the target:

```yaml
scrape_configs:
  - job_name: 'windows_exporter'
    static_configs:
      - targets: ['toyota-core.itrack.gr:9182']
        labels:
          instance: 'toyota-core'
```

Then restart Prometheus:

```bash
sudo systemctl restart prometheus
```

### Metrics Endpoint

```
http://<windows_host>:9182/metrics
```