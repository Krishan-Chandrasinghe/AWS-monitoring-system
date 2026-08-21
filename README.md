# AWS EC2 Infrastructure Monitoring Stack

A production-grade system monitoring solution built on AWS EC2 using Docker, Prometheus, Node Exporter, and Grafana with automated Discord Webhook Alerting.

---

## 📌 Architecture Overview
```
[ AWS EC2 Instance ]
  └── Node Exporter (Port 9100) ──> Scrapes host system metrics
         │
         ▼
    Prometheus (Port 9090)     ──> Stores time-series data
         │
         ▼
     Grafana (Port 3000)       ──> Visualizes data & evaluates alert rules
         │
         ▼
   Discord Webhook             ──> Delivers real-time Firing/Resolved alerts

┌────────────────────────────────────────────────────────────────────────┐
│                          AWS EC2 Instance                              │
│                                                                        │
│   ┌─────────────────┐        ┌───────────────┐      ┌──────────────┐   │
│   │  Node Exporter  │ ────>  │  Prometheus   │ ───> │   Grafana    │   │
│   │   (Port 9100)   │        │  (Port 9090)  │      │ (Port 3000)  │   │
│   └─────────────────┘        └───────────────┘      └──────┬───────┘   │
└────────────────────────────────────────────────────────────┼───────────┘
│                                                            ▼
▼                                                    ┌──────────────────┐
┌────────────────┐                                   │ Visualizes data  │
│ Discord Alert  │                                   │   & evaluates    │
│    Webhook     │                                   │   alert rules    │
└────────────────┘                                   └──────────────────┘
```
---

## 🚀 Tech Stack

* Cloud: AWS EC2 (Ubuntu)
* Containerization: Docker & Docker Compose
* Monitoring: Prometheus & Node Exporter
* Visualization: Grafana
* Alerting: Discord Webhooks

---

## 🛠️ System Requirements

Before you begin, ensure you have the following:

* An AWS EC2 Instance (Ubuntu 22.04 LTS or 24.04 LTS recommended)
* Docker and Docker Compose installed on the EC2 instance
* An active Discord account and server

---

## 📖 Step-by-Step Setup Guide

### Step 1: Configure AWS Security Group Rules

Allow inbound traffic for the monitoring services via your AWS EC2 Console:

1. Go to EC2 Console -> Instances -> Select your Instance.
2. Open the Security tab and click on the associated Security Group.
3. Click Edit inbound rules and add the following rules:

| Type | Port Range | Source | Purpose |
| :--- | :--- | :--- | :--- |
| Custom TCP | 9090 | 0.0.0.0/0 | Prometheus Dashboard |
| Custom TCP | 3000 | 0.0.0.0/0 | Grafana Dashboard |
| Custom TCP | 9100 | 0.0.0.0/0 | Node Exporter Metrics (Optional for testing) |

---

### Step 2: Set Up Discord Webhook

1. Open Discord and navigate to your target server (or create a new server).
2. Right-click on your alert channel (e.g., #alerts) and select Edit Channel (Gear Icon).
3. Go to Integrations -> Webhooks -> Click New Webhook.
4. Name the webhook (e.g., Grafana Alerts).
5. Click Copy Webhook URL and save it for later.

---

### Step 3: Clone Repository & Start Services

SSH into your EC2 server and run:

# Clone the repository
```
git clone https://github.com/Krishan-Chandrasinghe/AWS-monitoring-system.git
cd AWS-monitoring-system
```

# Spin up all containerized services in detached mode
```
docker compose up -d
```

# Verify that all three containers are running
```
docker ps
```

#### Step 3.1: Verify Running Services via Browser
To ensure all monitoring components are working properly, open your browser and test the following endpoints:

1. Node Exporter Metrics:
   Navigate to: http://<YOUR_EC2_PUBLIC_IP>:9100/metrics
   Expected Result: A raw text page containing raw system metrics (e.g., node_cpu_seconds_total).

2. Prometheus Web UI:
   Navigate to: http://<YOUR_EC2_PUBLIC_IP>:9090
   Expected Result: Prometheus UI loads. Go to Status -> Targets to ensure node-exporter status shows UP (1/1).

3. Grafana Dashboard:
   Navigate to: http://<YOUR_EC2_PUBLIC_IP>:3000
   Expected Result: Grafana login screen appears.

---

### Step 4: Configure Grafana Data Source

1. Open your browser and navigate to http://<YOUR_EC2_PUBLIC_IP>:3000.
2. Log in with the default credentials:
   * Username: admin
   * Password: admin (You will be prompted to set a new password).
3. Go to Connections -> Data sources -> Click Add data source.
4. Select Prometheus.
5. Set the Prometheus server URL to: http://prometheus:9090
6. Scroll down and click Save & test. You should see a green success notification.

---

### Step 5: Import Pre-built Dashboard

1. In Grafana, navigate to Dashboards -> Click New -> Select Import.
2. Click Upload dashboard JSON file and select the `node-exporter-dashboard.json` file provided in this repository.
   (Alternatively, you can import via Grafana.com by entering ID 1860 - Node Exporter Full and clicking Load).
3. Select your Prometheus Data Source from the dropdown.
4. Click Import.

---

### Step 6: Create Grafana Discord Contact Point

1. In Grafana, go to Alerting -> Contact points -> Click Add contact point.
2. Name: Discord Alerts
3. Integration: Select Discord.
4. Webhook URL: Paste your Discord Webhook URL created in Step 2.
5. Click Test to send a test notification to your Discord channel.
6. Click Save contact point.

---

### Step 7: Configure Alert Rules & Notifications

Go to Alerting -> Alert rules -> Create alert rule to set up system monitors:

#### 1. High CPU Usage Alert
* Query Setup:
  * Metric: node_cpu_seconds_total
  * Label filter: mode = idle
  * Operations: Rate (5m) -> Multiply by scalar (-100) -> Add scalar (100) -> Avg by (instance)
* Condition: IS ABOVE 80
* Evaluation Behavior: Interval: 1m | Pending period: 2m
* Configure Notifications: Under Step 5 (Configure notifications), set Contact point to `Discord Alerts`.

#### 2. High Memory Usage Alert
* Query Setup:
  * Metric: node_memory_MemAvailable_bytes
  * Operations: Binary Operation (/) -> node_memory_MemTotal_bytes -> Multiply by scalar (100)
* Condition: IS BELOW 20 (Triggers when free available memory drops below 20%)
* Evaluation Behavior: Interval: 1m | Pending period: 1m
* Configure Notifications: Under Step 5 (Configure notifications), set Contact point to `Discord Alerts`.

---

## 🧪 Testing System Alerts

To verify end-to-end alert delivery, simulate high CPU load directly on your EC2 instance:

# Install stress testing utility
```
sudo apt update && sudo apt install stress -y
```

# Generate 100% CPU load across 2 cores for 2 minutes
```
stress --cpu 2 --timeout 120s
```

Within 1-2 minutes, Grafana will trigger an alert and dispatch a [FIRING] notification to your Discord channel. Once the load normalizes, an automated [RESOLVED] notification will follow.

---

## 📄 License

This project is open-source and available under the MIT License.