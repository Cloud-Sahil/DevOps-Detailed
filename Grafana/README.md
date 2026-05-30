```
this file will contain all the steps needed for the promethous and grafana setup
```
## Prometheus & Grafana Setup Guide (Ubuntu 20.04/22.04)
## 1. Install Required Packages
```bash
sudo apt update && sudo apt install -y software-properties-common wget gnupg
```
## 2. Install Grafana (Official Repository)
```bash
wget -q -O - https://packages.grafana.com/gpg.key | \
sudo gpg --dearmor -o /usr/share/keyrings/grafana.key

echo "deb [signed-by=/usr/share/keyrings/grafana.key] https://packages.grafana.com/oss/deb stable main" | \
sudo tee /etc/apt/sources.list.d/grafana.list
```
### Install Grafana
```bash
sudo apt update
sudo apt install grafana -y
```
## Enable & Start Grafana
```bash
sudo systemctl enable --now grafana-server
```
Grafana Dashboard:
👉 http://<server-ip>:3000
Default login: admin / admin


## 3. Install Prometheus
Download Prometheus
```bash
cd /opt
sudo wget https://github.com/prometheus/prometheus/releases/download/v2.52.0/prometheus-2.52.0.linux-amd64.tar.gz
sudo tar -xvf prometheus-2.52.0.linux-amd64.tar.gz
sudo mv prometheus-2.52.0.linux-amd64 prometheus
```
Create Prometheus User & Directories
```bash
sudo useradd --no-create-home --shell /usr/sbin/nologin prometheus
sudo mkdir /etc/prometheus /var/lib/prometheus
sudo chown prometheus:prometheus /etc/prometheus /var/lib/prometheus
```

Move Prometheus Binaries
```bash
sudo cp /opt/prometheus/prometheus /usr/local/bin/
sudo cp /opt/prometheus/promtool /usr/local/bin/
```

Copy consoles & libraries
```bash
sudo cp -r /opt/prometheus/consoles /etc/prometheus/
sudo cp -r /opt/prometheus/console_libraries /etc/prometheus/
```
## 4. Create Prometheus Config File
```bash
sudo vim /etc/prometheus/prometheus.yml
```
```yaml
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "node_exporter"
    static_configs:
      - targets: ["localhost:9100"]
```
## 5. Create Prometheus Systemd Service
```bash
sudo nano /etc/systemd/system/prometheus.service
```
```
[Unit]
Description=Prometheus Monitoring
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus/ \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```
## Reload & Start Prometheus
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now prometheus
sudo systemctl status prometheus
```
Prometheus UI:
👉 http://<server-ip>:9090


## 6. Install Node Exporter
Download Node Exporter
```bash
cd /opt
sudo wget https://github.com/prometheus/node_exporter/releases/download/v1.8.1/node_exporter-1.8.1.linux-amd64.tar.gz
sudo tar -xvf node_exporter-1.8.1.linux-amd64.tar.gz
sudo mv node_exporter-1.8.1.linux-amd64 node_exporter
```
Create User
```bash
sudo useradd --no-create-home --shell /usr/sbin/nologin nodeusr
```
Move Binary & Set Ownership
```bash
sudo cp /opt/node_exporter/node_exporter /usr/local/bin/
sudo chown nodeusr:nodeusr /usr/local/bin/node_exporter
```
## 7. Create Node Exporter Service
```bash
sudo nano /etc/systemd/system/node_exporter.service
```
```
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=nodeusr
Group=nodeusr
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```
## Reload & Start Node Exporter
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now node_exporter
sudo systemctl status node_exporter
```
Node Exporter Metrics:
👉 http://<server-ip>:9100/metrics

import a ready-made Grafana dashboard.
Node Exporter Full Dashboard ID → 1860
Just paste this number in the box:
👉 1860

want to add trigers 
```bash
vim /etc/prometheus/prometheus.yml 
```
<img src="./images/localhost_file.png" width="400">

## steps to setup grafana grafacally after installation 
navigate to promethious 
check for entry of your dashboard 

<img src="./images/navigate_to_prometheous_to_add.png" width="600">

<img src="./images/select_target_to_see.png" width="600">

 you will see all the dashboard which are created in your prometheus dashboard 
![dashboard](./images/promethes_dashboard.png)

## on grafana dashboard 
add new connection 
![new](./images/add_new_data_source_in_grafana.png)
![add](./images/add_promethious_to_grafana.png)

create new data source 
![creat](./images/create_connection.png)


navigate to grafana dashboard and import new dashboard 
![newdashboardimport](./images/import_new_dashboard.png)

now create new dashboard by updating an Node Exporter full 
![update](./images/import_dashboard.png)

dashboard is here 
![dashboard](./images/grafana_dashboard.png)


## for creating an multipal dashboard repeart the promethious and grafana steps change the name and id in dashboard adding 


create new data source 
![creat](./images/create_connection.png)


navigate to grafana dashboard and import new dashboard 
![newdashboardimport](./images/import_new_dashboard.png)

now create new dashboard by updating an Node Exporter full 
![update](./images/import_dashboard.png)


## Dashboard is here with multipal IP dashboard is heare 
![multipal_ip_dashboard](./images/multipal_dashboard.png)





# Grafana Dashboard IDs 🔥

| Dashboard            | ID        | Description                  |
| -------------------- | --------- | ---------------------------- |
| Node Exporter Full   | **1860**  | Full Linux system monitoring |
| Prometheus 2.0 Stats | **3662**  | Prometheus internal metrics  |
| Docker Monitoring    | **193**   | Container metrics            |
| System Monitoring    | **10502** | CPU, RAM, Disk, Network      |




