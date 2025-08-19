# Monitoring Lab with Prometheus, Grafana & Node Exporter

This repo documents my hands on setup of a **monitoring stack** across multiple laptops (Linux + Windows).  
The goal: learn real-world monitoring, practice troubleshooting, and build something I can showcase & will later use on my own HomeLab



## Overview

- **Prometheus** → collects metrics  
- **Grafana** → visualizes metrics with dashboards  
- **Node Exporter (Linux)** → system metrics (CPU, RAM, Disk, etc.)  
- **Windows Exporter** → system metrics for Windows  



## Architecture

```diff
+ [Laptop 1 - ProBook]
 ├── Prometheus (192.168.x.x:9090)
 ├── Grafana    (192.168.x.x:3000)
 └── Node Exporter (192.168.x.x:9100)

+ [Laptop 2 - Linux Secondary]
 └── Node Exporter (192.168.x.x:9100)

+ [Windows Desktop]
 └── Windows Exporter (192.168.x.x:9182)
```

## Installation Steps
# 1. Prometheus
```
wget https://github.com/prometheus/prometheus/releases/download/vx.x.x/prometheus-x.x.x.linux-amd64.tar.gz
tar -xvzf prometheus-*.tar.gz
sudo mv prometheus-*/prometheus /usr/local/bin/
```

Create service file (/etc/systemd/system/prometheus.service):
```diff
- [Unit]

Description=Prometheus
After=network.target

- [Service]
User=xxxx
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus
Restart=always

- [Install]
WantedBy=multi-user.target
```
# Config Example
```
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "linux"
    static_configs:
      - targets: ["192.168.x.x:9100", "192.168.x.x:9100"]

  - job_name: + "windows"
    static_configs:
      - targets: ["192.168.x.x:9182"]
```

# 2. Grafana

Installed via .tar.gz, configured service under /etc/systemd/system/grafana.service.

# 3. Node Exporter (Linux)

Installed on every device I wanted to monitor, in this example 2x Linux Laptops + 1x Windows PC
```
wget https://github.com/prometheus/node_exporter/releases/download/vx.x.x/node_exporter-x.x.x.linux-amd64.tar.gz
tar -xvzf node_exporter-*.tar.gz
sudo mv node_exporter-*/node_exporter /usr/local/bin/
```
# Systemd service:
```diff
- [Unit]
Description=Node Exporter
After=network.target

- [Service]
ExecStart=/usr/local/bin/node_exporter
Restart=always

- [Install]
WantedBy=multi-user.target
```

# 4. Windows Exporter

Downloaded from https://github.com/prometheus-community/windows_exporter/releases.
Runs as a Windows Service on port 9182.

⚠️ Still debugging: target currently shows context deadline exceeded.
(Troubleshooting steps included below.)

--

I hit multiple issues along the way:

❌ Prometheus service failed → fixed by correcting User= in service file.

❌ Grafana failed to start → wrong config path, fixed with correct permissions.

❌ Targets UNKNOWN → solved by adding correct static IPs & firewall rules.

❌ Windows Exporter DOWN → still investigating (possibly firewall/Norton).


---

## Troubleshooting 

| Issue | Cause | Solution Tried | Result / Notes |
|-------|-------|----------------|----------------|
| Prometheus service failed to start | Wrong user in service file | Corrected `User=` in `/etc/systemd/system/prometheus.service` | ✅ Fixed, service active |
| Grafana service inactive | Wrong config path + permissions | Fixed paths, set correct permissions, enabled service | ✅ Fixed |
| Targets all showed UNKNOWN | Dynamic IPs changed after reboot | Changed to **static IPs** in `/etc/netplan/` | ✅ Fixed |
| Prometheus reload failed (`reload is not a type`) | Wrong command used | Restarted service with `systemctl restart prometheus` | ✅ Fixed |
| Windows target shows `context deadline exceeded` | Connection timeout | - Opened port `9182` in Norton & Windows Firewall<br>- Tried inbound + outbound rules<br>- Disabled Norton completely | ❌ Still DOWN, likely deeper firewall/routing issue |
| Couldn’t ping Windows PC | ICMP blocked | Added firewall rule for ICMP in Norton + Windows Firewall | ❌ Still failing |
| Services disappeared in Norton firewall | Rules not persisting | Recreated rules manually | Still unstable |
| UFW blocking exporters | Default `deny in` rule in UFW | Allowed ports (`sudo ufw allow 9090,9100,3000/tcp`) | ✅ Linux targets now UP |

---

## 🔐 Firewall & Security Notes

### 🔹 What I Did
- Allowed ports in UFW individually (`9090`, `9100`, `3000`)  
- Later tested with `sudo ufw allow from any` to eliminate blocking as a variable  
- On Windows, created inbound + outbound rules for port `9182` (Windows Exporter)  
- Disabled Norton temporarily to check if it was the issue (was not the only problem)  

### 🔹 Best Practice
- Use **UFW** with explicit rules per service, not “allow all”  
- Limit Prometheus & Grafana ports (`9090`, `3000`) to LAN only:  
  ```bash
  sudo ufw allow from 192.168.0.0/24 to any port 9090
  sudo ufw allow from 192.168.0.0/24 to any port 3000
  ```

Keep 9100 (Linux Node Exporter) and 9182 (Windows Exporter) open only to Prometheus server

Avoid disabling AV/firewall entirely; create granular rules instead

Document all exceptions so they don’t get lost (like with Norton)












This repo doesn’t just show success, it shows problem solving.


To make it visual & impressive, I’ll upload these screenshots:
--
**Prometheus targets page** (http://localhost:9090/targets)

**Grafana dashboard with system metrics**

**Linux Node Exporter metrics page** (http://localhost:9100/metrics)

**Windows Exporter page attempt** (http://192.168.x.x:9182/metrics)

*Any errors fixed (like service status logs)*

---

## Follow Me on Social Media

Stay connected and follow my work on social media:

- **Main 𝕏 Account:** [FunkyxBeatz](https://x.com/FunkyxBeatz)
- **Projects 𝕏 Account:** [WebFrens_](https://x.com/WebFrens_)





































