# Prometheus_grafana_setup_python_app
o:

````markdown
# 📊 Monitoring Stack with Prometheus, Grafana & Python App

This project demonstrates how to set up a complete monitoring stack on an **Amazon Linux EC2 server** using **Prometheus, Grafana, and Node Exporter**, with integration of a custom **Python Flask app** exposing Prometheus metrics. It also includes alerting rules via `rules.yml`.

---

## 🚀 Components

- **Prometheus** – Metrics collection & alerting  
- **Grafana** – Visualization and dashboards  
- **Node Exporter** – System metrics exporter (CPU, Memory, Disk, Load)  
- **Python Flask App** – Custom app exposing `/metrics` endpoint  
- **Alertmanager** – (configured in Prometheus for handling alerts)  
- **rules.yml** – Prometheus alerting rules (e.g., high CPU usage)  

---

## ⚙️ Installation Steps

### 1️⃣ Install Prometheus
```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.55.1/prometheus-2.55.1.linux-amd64.tar.gz
tar xvf prometheus-*.tar.gz
cd prometheus-*
````

Update config file:

```yaml
# /etc/prometheus/prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets: ["alertmanager:9093"]

rule_files:
  - "rules.yml"

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "node"
    static_configs:
      - targets: ["localhost:9100"]

  - job_name: "app"
    static_configs:
      - targets: ["localhost:5000"]
```

Start Prometheus:

```bash
./prometheus --config.file=prometheus.yml
```

---

### 2️⃣ Install Node Exporter

```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.1/node_exporter-1.8.1.linux-amd64.tar.gz
tar xvf node_exporter-*.tar.gz
cd node_exporter-*
./node_exporter
```

Node Exporter runs by default on **:9100**.

---

### 3️⃣ Python Flask App with Prometheus Client

Install dependencies:

```bash
sudo dnf install -y python3-pip
pip3 install flask prometheus-client
```

App code (`app.py`):

```python
from prometheus_client import Counter, generate_latest, CONTENT_TYPE_LATEST
from flask import Flask

app = Flask(__name__)
REQUESTS = Counter('hello_worlds_total','Hello Worlds requested.')

@app.route('/')
def hello_world():
    REQUESTS.inc()
    return 'Hello, World!'

@app.route('/metrics')
def metrics():
    return generate_latest(), 200, {'Content-Type': CONTENT_TYPE_LATEST}

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=True)
```

Run:

```bash
python3 app.py
```

Check:

```bash
curl http://localhost:5000/
curl http://localhost:5000/metrics
```

---

### 4️⃣ Alerting Rules

`rules.yml` example:

```yaml
groups:
- name: example-group
  rules:
  - alert: HighCPUUsage
    expr: 100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "High CPU Usage detected (instance {{ $labels.instance }})"
      description: "CPU usage is above 80% for 1 minute on instance {{ $labels.instance }}"
```

---

### 5️⃣ Install Grafana

```bash
sudo dnf install -y grafana
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```

Access Grafana:
👉 `http://<EC2-Public-IP>:3000`

Default login:

* **Username:** `admin`
* **Password:** `admin` (change at first login)

---

### 6️⃣ Connect Prometheus to Grafana

1. Go to **Connections → Data sources → Add data source**
2. Choose **Prometheus**
3. Set URL:

   ```
   http://<EC2-Public-IP>:9090/
   ```
4. Click **Save & Test**

---

### 7️⃣ Create Dashboards

#### CPU Usage %

```promql
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[1m])) * 100)
```

#### Memory Usage %

```promql
(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100
```

#### Load Average

```promql
node_load1
```

#### Python App Requests per Second

```promql
rate(hello_worlds_total[1m])
```

---

## ✅ Testing

Run stress test:

```bash
sudo yum install -y stress
stress --cpu 2 --timeout 60s
```

* CPU panel in Grafana should spike.
* Python app counters (`hello_worlds_total`) increase when hitting `/`.

---

## 📌 Summary

* Prometheus scrapes **system + app metrics**
* Grafana visualizes **CPU, memory, load, app counters**
* Node Exporter provides **server metrics**
* Flask app exposes **custom metrics**
* rules.yml defines **alerts**

This setup provides a full **monitoring + alerting + visualization** pipeline.

```

```
