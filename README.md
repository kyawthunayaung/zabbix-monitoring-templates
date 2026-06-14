# Enterprise Zabbix 7.0+ Server Installation & Infrastructure Monitoring Blueprint

This repository contains production-ready installation playbooks, configuration files, and architecture mappings for deploying an enterprise-grade **Zabbix Monitoring Server** on Ubuntu LTS. It includes setups for optimized Web Frontends (running on custom ports like `8080`), database tuning for PostgreSQL, and automated notification triggers.

---

## 📊 Monitoring Architecture Overview
* **Zabbix Core Server:** Monitors host daemon availability via the native **ZBX** agent interface.
* **Database Backend:** PostgreSQL (with TimescaleDB extension for time-series optimization).
* **Web Engine:** Nginx / Apache mapped to custom ports (e.g., `8080/zabbix`) for environment isolation.
* **Alerting Framework:** Media types mapped to **Office 365 SMTP, Telegram, and Slack webhooks** with dual-state notification topologies (Problem & Recovery triggers).

---

## 🛠️ Step 1: Repository Provisioning & Core Installation
Configure official Zabbix upstream release repositories and install the Zabbix server stack components along with the PostgreSQL database schema files.

```bash
# Download and install the Zabbix official repository components
wget [https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.0+ubuntu24.04_amd64.deb](https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.0+ubuntu24.04_amd64.deb)
sudo dpkg -i zabbix-release_latest_7.0+ubuntu24.04_amd64.deb
sudo apt update

# Install Zabbix Server, Frontend, Agent, and SQL Schema packages
sudo apt install -y zabbix-server-pgsql zabbix-frontend-php zabbix-nginx-conf zabbix-sql-scripts zabbix-agent


🗄️Step 2: Database Initialization & Schema Import

# Log into PostgreSQL instance
sudo -u postgres psql

# Execute Database Hardening Queries
CREATE USER zabbix WITH PASSWORD 'your_secure_db_password_here';
CREATE DATABASE zabbix OWNER zabbix;
\q

# Decompress and import the initial server database schema script
zcat /usr/share/zabbix-sql-scripts/postgresql/server.sql.gz | sudo -u zabbix psql zabbix

⚙️ Step 3: Core Daemon Configuration (/etc/zabbix/zabbix_server.conf)

LogFile=/var/log/zabbix/zabbix_server.log
LogFileSize=10
PidFile=/var/run/zabbix/zabbix_server.pid
SocketDir=/var/run/zabbix
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=your_secure_db_password_here

🌐 Step 4: Web Frontend Port Mapping Optimization
-Configure the Nginx/Apache configuration virtual hosts block (/etc/zabbix/nginx.conf or equivalent):

server {
    listen 8080;
    server_name monitoring.your-domain.net;

    root /usr/share/zabbix;

    location /zabbix {
        index index.php;
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php-fpm.sock; # Adjust PHP-FPM version string accordingly
    }
}

-Enable and start all automated daemons on system init runtime:

sudo systemctl restart zabbix-server zabbix-agent nginx php-fpm
sudo systemctl enable zabbix-server zabbix-agent nginx php-fpm

🎛️ Step 5: Enterprise Alerting & Severity Mappings

The system utilizes native templates matching the global color-coded standard matrix to classify real-time anomalies detected within the Problems Widget Dashboard:

System Status Severity,  Hex Color Mapping,  Trigger Threshold Action
OK / Normal              #00FF00 (Green)     Clear state; System functions normally.
Warning                  #FFFF00 (Yellow)    Minor anomaly detected; Proactive triage monitoring.
Average                  #FFA500 (Orange)    Performance or network bottleneck mitigation required.
High / Disaster          #FF0000 (Red)       Critical outage; Immediate engineer on-call page required.

🚨 Native Notifications Integration Framework

Configure Zabbix Actions (Alerts > Actions > Trigger Actions) to build a high-availability alert distribution cycle:
1-Problem Initiation Payload: Sent immediately via SMTP/Webhook upon condition match.
2-Acknowledgement Mechanism (Ack): Engineers flags active status on the UI dashboard (Ack = Yes) to eliminate overlapping team triage workflows.
3-Recovery Messages Protocol: Enforces an automated follow-up RESOLVED status notification as soon as the target item metric clears the problem state.

💡 Maintained by Kyaw Thu Nay Aung — Systems & Infrastructure Engineering Specialist.


