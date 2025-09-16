# Shinobi Installation Guide

Shinobi is a free and open-source CCTV/NVR (Network Video Recorder) solution written in Node.js. It serves as a modern FOSS alternative to proprietary video surveillance systems like Blue Iris, Milestone XProtect, Genetec Security Center, or commercial NVR solutions from Hikvision, Dahua, and others. Shinobi supports RTSP, ONVIF, and various IP camera protocols with features including motion detection, object detection, and multi-user support.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

### Hardware Requirements
- **CPU**: Minimum 2 cores (4+ cores recommended for multiple cameras)
- **RAM**: 4GB minimum (8GB+ recommended)
- **Storage**: 100GB+ for recordings (calculate based on camera count and retention)
- **GPU**: Optional but recommended for hardware acceleration
- **Network**: Gigabit Ethernet recommended for multiple HD cameras

### Software Requirements
- **Node.js**: 14.x or later
- **FFmpeg**: 4.x or later with x264/x265 support
- **MariaDB/MySQL**: 5.7+ or MariaDB 10.3+
- **Git**: For cloning repository
- **Python3**: For some plugins

### Network Requirements
- **Ports**: 
  - 8080: Web interface (HTTP)
  - 8443: Web interface (HTTPS)
  - 21: FTP (optional)
  - Camera ports (varies by manufacturer)
- **Firewall**: Allow camera network access
- **Bandwidth**: 2-8 Mbps per camera (varies by resolution)

## 2. Supported Operating Systems

Shinobi officially supports:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04 LTS / 22.04 LTS / 24.04 LTS
- Arch Linux
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- macOS 12+ (via Homebrew)
- Windows 10/11 (via WSL2 or native)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Enable EPEL and PowerTools/CRB repositories
sudo dnf install -y epel-release
sudo dnf config-manager --set-enabled powertools  # CentOS Stream 8
sudo dnf config-manager --set-enabled crb        # RHEL/Rocky/Alma 9

# Install dependencies
sudo dnf install -y ffmpeg ffmpeg-devel x264 x265 x264-devel x265-devel \
    git curl gnupg2 gcc-c++ make cairo-devel jpeg-devel \
    giflib-devel pango-devel python3 python3-pip

# Install Node.js 18.x
curl -fsSL https://rpm.nodesource.com/setup_18.x | sudo bash -
sudo dnf install -y nodejs

# Install MariaDB
sudo dnf install -y mariadb mariadb-server
sudo systemctl enable --now mariadb
sudo mysql_secure_installation

# Create Shinobi database and user
sudo mysql -e "CREATE DATABASE ccio;"
sudo mysql -e "CREATE USER 'ccio'@'localhost' IDENTIFIED BY 'your_secure_password';"
sudo mysql -e "GRANT ALL PRIVILEGES ON ccio.* TO 'ccio'@'localhost';"
sudo mysql -e "FLUSH PRIVILEGES;"

# Clone and install Shinobi
cd /opt
sudo git clone https://gitlab.com/Shinobi-Systems/Shinobi.git shinobi
sudo chown -R $USER:$USER /opt/shinobi
cd /opt/shinobi

# Install Node.js dependencies
npm install --unsafe-perm
npm install pm2 -g

# Configure Shinobi
cp conf.sample.json conf.json
cp super.sample.json super.json

# Initialize database
node /opt/shinobi/sql/framework.js

# Create systemd service (see Service Management section)
```

### Debian/Ubuntu

```bash
# Update package lists
sudo apt update

# Install FFmpeg and multimedia libraries
sudo apt install -y ffmpeg x264 x265 git curl gnupg2 \
    build-essential python3 python3-pip libcairo2-dev \
    libjpeg-dev libgif-dev libpango1.0-dev

# Install Node.js 18.x
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

# Install MariaDB
sudo apt install -y mariadb-server mariadb-client
sudo systemctl enable --now mariadb
sudo mysql_secure_installation

# Create Shinobi database and user
sudo mysql -e "CREATE DATABASE ccio;"
sudo mysql -e "CREATE USER 'ccio'@'localhost' IDENTIFIED BY 'your_secure_password';"
sudo mysql -e "GRANT ALL PRIVILEGES ON ccio.* TO 'ccio'@'localhost';"
sudo mysql -e "FLUSH PRIVILEGES;"

# Clone and install Shinobi
cd /opt
sudo git clone https://gitlab.com/Shinobi-Systems/Shinobi.git shinobi
sudo chown -R $USER:$USER /opt/shinobi
cd /opt/shinobi

# Install dependencies
npm install --unsafe-perm
sudo npm install pm2 -g

# Configure Shinobi
cp conf.sample.json conf.json
cp super.sample.json super.json

# Initialize database
node /opt/shinobi/sql/framework.js
```

### Arch Linux

```bash
# Update system
sudo pacman -Syu

# Install dependencies
sudo pacman -S ffmpeg x264 x265 git curl gnupg nodejs npm \
    python python-pip base-devel cairo libjpeg-turbo \
    libpng pango giflib mariadb

# Enable and start MariaDB
sudo systemctl enable --now mariadb
sudo mysql_secure_installation

# Create Shinobi database and user
sudo mysql -e "CREATE DATABASE ccio;"
sudo mysql -e "CREATE USER 'ccio'@'localhost' IDENTIFIED BY 'your_secure_password';"
sudo mysql -e "GRANT ALL PRIVILEGES ON ccio.* TO 'ccio'@'localhost';"
sudo mysql -e "FLUSH PRIVILEGES;"

# Clone and install Shinobi
cd /opt
sudo git clone https://gitlab.com/Shinobi-Systems/Shinobi.git shinobi
sudo chown -R $USER:$USER /opt/shinobi
cd /opt/shinobi

# Install dependencies
npm install --unsafe-perm
sudo npm install pm2 -g

# Configure Shinobi
cp conf.sample.json conf.json
cp super.sample.json super.json

# Initialize database
node /opt/shinobi/sql/framework.js
```

### Alpine Linux

```bash
# Update package index
sudo apk update

# Install dependencies
sudo apk add --no-cache ffmpeg x264 x265 git curl gnupg \
    nodejs npm python3 py3-pip build-base cairo-dev \
    jpeg-dev libpng-dev pango-dev giflib-dev mariadb \
    mariadb-client mariadb-dev

# Initialize MariaDB
sudo mysql_install_db --user=mysql --datadir=/var/lib/mysql
sudo rc-service mariadb start
sudo rc-update add mariadb default
sudo mysql_secure_installation

# Create Shinobi database and user
sudo mysql -e "CREATE DATABASE ccio;"
sudo mysql -e "CREATE USER 'ccio'@'localhost' IDENTIFIED BY 'your_secure_password';"
sudo mysql -e "GRANT ALL PRIVILEGES ON ccio.* TO 'ccio'@'localhost';"
sudo mysql -e "FLUSH PRIVILEGES;"

# Clone and install Shinobi
cd /opt
sudo git clone https://gitlab.com/Shinobi-Systems/Shinobi.git shinobi
sudo chown -R $USER:$USER /opt/shinobi
cd /opt/shinobi

# Install dependencies
npm install --unsafe-perm
sudo npm install pm2 -g

# Configure Shinobi
cp conf.sample.json conf.json
cp super.sample.json super.json

# Initialize database
node /opt/shinobi/sql/framework.js
```

## 4. Configuration

### Database Configuration

Edit `/opt/shinobi/conf.json`:
```json
{
  "port": 8080,
  "addStorage": [
    {
      "name": "second",
      "path": "/opt/shinobi/videos2"
    }
  ],
  "ssl": {
    "key": "/path/to/private.key",
    "cert": "/path/to/certificate.crt",
    "enabled": false,
    "port": 8443
  },
  "db": {
    "host": "127.0.0.1",
    "user": "ccio",
    "password": "your_secure_password",
    "database": "ccio",
    "port": 3306
  },
  "mail": {
    "auth": {
      "user": "your_email@example.com",
      "pass": "your_app_password"
    },
    "host": "smtp.gmail.com",
    "port": 587,
    "secure": false
  },
  "cron": {
    "deleteOld": "0 * * * *"
  }
}
```

### Super User Configuration

Edit `/opt/shinobi/super.json`:
```json
{
  "port": 8080,
  "bindip": "0.0.0.0",
  "ssl": {
    "key": "",
    "cert": "",
    "enabled": false,
    "port": 8443
  },
  "db": {
    "host": "127.0.0.1",
    "user": "ccio",
    "password": "your_secure_password",
    "database": "ccio",
    "port": 3306
  },
  "mail": {
    "auth": {
      "user": "your_email@example.com",
      "pass": "your_app_password"
    },
    "host": "smtp.gmail.com",
    "port": 587,
    "secure": false
  }
}
```

### Creating Storage Directories

```bash
# Create video storage directories
sudo mkdir -p /opt/shinobi/videos
sudo mkdir -p /var/shinobi/recordings
sudo chown -R shinobi:shinobi /opt/shinobi/videos
sudo chown -R shinobi:shinobi /var/shinobi/recordings

# Set proper permissions
sudo chmod 755 /opt/shinobi/videos
sudo chmod 755 /var/shinobi/recordings
```

## 5. Service Management

### Create Shinobi User

```bash
# Create dedicated user
sudo useradd -r -s /bin/false -d /opt/shinobi shinobi
sudo chown -R shinobi:shinobi /opt/shinobi
```

### systemd Service (RHEL, Debian, Arch, SUSE)

Create `/etc/systemd/system/shinobi.service`:
```ini
[Unit]
Description=Shinobi CCTV System
After=network.target mariadb.service
Wants=mariadb.service

[Service]
Type=forking
User=shinobi
Group=shinobi
WorkingDirectory=/opt/shinobi
ExecStart=/usr/bin/pm2 start ecosystem.config.js --env production
ExecReload=/usr/bin/pm2 reload ecosystem.config.js --env production
ExecStop=/usr/bin/pm2 stop ecosystem.config.js
PIDFile=/opt/shinobi/.pm2/pm2.pid
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

Create `/opt/shinobi/ecosystem.config.js`:
```javascript
module.exports = {
  apps: [
    {
      name: 'camera',
      script: 'camera.js',
      cwd: '/opt/shinobi',
      instances: 1,
      exec_mode: 'fork',
      env: {
        NODE_ENV: 'production'
      }
    },
    {
      name: 'cron',
      script: 'cron.js',
      cwd: '/opt/shinobi',
      instances: 1,
      exec_mode: 'fork',
      env: {
        NODE_ENV: 'production'
      }
    }
  ]
};
```

```bash
# Enable and start services
sudo systemctl daemon-reload
sudo systemctl enable --now shinobi
sudo systemctl status shinobi
```

### OpenRC Service (Alpine Linux)

Create `/etc/init.d/shinobi`:
```bash
#!/sbin/openrc-run

description="Shinobi CCTV System"
command="/usr/bin/pm2"
command_args="start /opt/shinobi/ecosystem.config.js --env production"
command_user="shinobi"
pidfile="/run/shinobi.pid"

depend() {
    need net mariadb
    after mariadb
}

start_pre() {
    checkpath --directory --owner shinobi:shinobi --mode 0755 /run/shinobi
}
```

```bash
# Make executable and enable
sudo chmod +x /etc/init.d/shinobi
sudo rc-update add shinobi default
sudo rc-service shinobi start
```

## 6. Troubleshooting

### Common Issues

1. **Shinobi won't start**:
```bash
# Check logs
sudo journalctl -u shinobi -f

# Check PM2 status
sudo -u shinobi pm2 status
sudo -u shinobi pm2 logs

# Check database connection
mysql -u ccio -p ccio -e "SHOW TABLES;"
```

2. **Cameras not connecting**:
```bash
# Test RTSP stream manually
ffmpeg -rtsp_transport tcp -i rtsp://username:password@camera_ip:554/stream -t 10 test.mp4

# Check network connectivity
ping camera_ip
telnet camera_ip 554
```

3. **High CPU usage**:
```bash
# Check encoding settings in monitor configuration
# Reduce video quality or frame rate
# Enable hardware acceleration if available
```

4. **Storage issues**:
```bash
# Check disk space
df -h /opt/shinobi/videos

# Check permissions
ls -la /opt/shinobi/videos

# Check cron job for cleanup
sudo -u shinobi crontab -l
```

## 7. Security Considerations

### SSL/TLS Configuration

```bash
# Generate self-signed certificate
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /opt/shinobi/ssl/shinobi.key \
    -out /opt/shinobi/ssl/shinobi.crt

# Set permissions
sudo chown shinobi:shinobi /opt/shinobi/ssl/*
sudo chmod 600 /opt/shinobi/ssl/shinobi.key
sudo chmod 644 /opt/shinobi/ssl/shinobi.crt
```

Update `/opt/shinobi/conf.json`:
```json
{
  "ssl": {
    "key": "/opt/shinobi/ssl/shinobi.key",
    "cert": "/opt/shinobi/ssl/shinobi.crt",
    "enabled": true,
    "port": 8443
  }
}
```

### Firewall Configuration

#### firewalld (RHEL/CentOS)
```bash
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --permanent --add-port=8443/tcp
sudo firewall-cmd --reload
```

#### ufw (Ubuntu/Debian)
```bash
sudo ufw allow 8080/tcp
sudo ufw allow 8443/tcp
sudo ufw enable
```

#### iptables
```bash
sudo iptables -A INPUT -p tcp --dport 8080 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 8443 -j ACCEPT
sudo iptables-save | sudo tee /etc/iptables/rules.v4
```

### User Authentication

1. **Change default admin password**:
   - Login to web interface: http://your_server:8080
   - Go to Settings → Users
   - Change admin password

2. **Create limited users**:
   - Add users with specific camera access
   - Set view-only permissions for monitors

3. **API Key Security**:
```bash
# Generate secure API keys
openssl rand -hex 32

# Rotate keys regularly via web interface
```

## 8. Performance Tuning

### Hardware Acceleration

Enable GPU acceleration in `/opt/shinobi/conf.json`:
```json
{
  "accelerator": {
    "enabled": true,
    "hwaccel": "vaapi",
    "hwaccel_device": "/dev/dri/renderD128"
  }
}
```

### CPU Optimization

```bash
# Set CPU governor to performance
echo performance | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor

# Adjust process priorities
sudo systemctl edit shinobi
```

Add to override file:
```ini
[Service]
Nice=-10
IOSchedulingClass=1
IOSchedulingPriority=4
```

### Memory Optimization

```bash
# Increase shared memory for large installations
echo 'kernel.shmmax = 268435456' | sudo tee -a /etc/sysctl.conf
echo 'kernel.shmall = 2097152' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

### Storage Optimization

```bash
# Use SSD for database and temporary files
sudo mkdir -p /var/lib/mysql-ssd
sudo mount /dev/nvme0n1p1 /var/lib/mysql-ssd
sudo chown mysql:mysql /var/lib/mysql-ssd

# Configure log rotation
sudo nano /etc/logrotate.d/shinobi
```

Add:
```
/opt/shinobi/logs/*.log {
    daily
    missingok
    rotate 7
    compress
    delaycompress
    copytruncate
}
```

## 9. Backup and Restore

### Database Backup

```bash
#!/bin/bash
# /opt/shinobi/scripts/backup-db.sh

BACKUP_DIR="/var/backups/shinobi"
DATE=$(date +%Y%m%d_%H%M%S)
DB_USER="ccio"
DB_PASS="your_secure_password"
DB_NAME="ccio"

# Create backup directory
mkdir -p $BACKUP_DIR

# Backup database
mysqldump -u $DB_USER -p$DB_PASS $DB_NAME > $BACKUP_DIR/shinobi_db_$DATE.sql

# Compress backup
gzip $BACKUP_DIR/shinobi_db_$DATE.sql

# Remove backups older than 30 days
find $BACKUP_DIR -name "shinobi_db_*.sql.gz" -mtime +30 -delete

echo "Database backup completed: $BACKUP_DIR/shinobi_db_$DATE.sql.gz"
```

### Configuration Backup

```bash
#!/bin/bash
# /opt/shinobi/scripts/backup-config.sh

BACKUP_DIR="/var/backups/shinobi"
DATE=$(date +%Y%m%d_%H%M%S)

# Create backup directory
mkdir -p $BACKUP_DIR

# Backup configuration files
tar -czf $BACKUP_DIR/shinobi_config_$DATE.tar.gz \
    /opt/shinobi/conf.json \
    /opt/shinobi/super.json \
    /opt/shinobi/ssl/ \
    /etc/systemd/system/shinobi.service

echo "Configuration backup completed: $BACKUP_DIR/shinobi_config_$DATE.tar.gz"
```

### Automated Backup Setup

```bash
# Add to crontab
sudo crontab -e

# Daily database backup at 2 AM
0 2 * * * /opt/shinobi/scripts/backup-db.sh

# Weekly configuration backup
0 3 * * 0 /opt/shinobi/scripts/backup-config.sh
```

### Restore Procedures

```bash
# Restore database
gunzip -c /var/backups/shinobi/shinobi_db_YYYYMMDD_HHMMSS.sql.gz | \
    mysql -u ccio -p ccio

# Restore configuration
tar -xzf /var/backups/shinobi/shinobi_config_YYYYMMDD_HHMMSS.tar.gz -C /

# Restart services
sudo systemctl restart mariadb
sudo systemctl restart shinobi
```

## 10. System Requirements

### Minimum Requirements
- **CPU**: 2 cores, 2.0 GHz
- **RAM**: 4GB
- **Storage**: 100GB (for 1-2 cameras)
- **Network**: 100 Mbps

### Recommended Requirements
- **CPU**: 4+ cores, 3.0+ GHz (or hardware acceleration)
- **RAM**: 8GB+
- **Storage**: 1TB+ SSD/NVMe
- **Network**: Gigabit Ethernet

### Scaling Guidelines
- **Per camera**: ~1GB RAM, 0.5 CPU cores
- **Storage**: 1-10GB per camera per day (varies by quality)
- **Bandwidth**: 2-8 Mbps per camera

## 11. Support

### Official Resources
- **GitLab**: https://gitlab.com/Shinobi-Systems/Shinobi
- **Documentation**: https://shinobi.video/docs
- **Community Forum**: https://community.shinobi.video
- **Discord**: https://discord.gg/mdhmvuH

### Troubleshooting Resources
- **Wiki**: https://gitlab.com/Shinobi-Systems/Shinobi/-/wikis/home
- **Issues**: https://gitlab.com/Shinobi-Systems/Shinobi/-/issues
- **Docker Hub**: https://hub.docker.com/r/shinobisystems/shinobi

## 12. Contributing

### How to Contribute
1. Fork the repository on GitLab
2. Create a feature branch
3. Submit merge request
4. Follow coding standards
5. Include tests and documentation

### Development Setup
```bash
# Clone development version
git clone https://gitlab.com/Shinobi-Systems/Shinobi.git
cd Shinobi

# Install development dependencies
npm install --dev

# Run in development mode
npm run dev
```

## 13. License

Shinobi is licensed under the GNU General Public License v3.0 (GPLv3).

Key points:
- Free to use, modify, and distribute
- Source code must remain open
- Commercial use allowed
- No warranty provided

## 14. Acknowledgments

### Credits
- **Moe Alam**: Creator and primary developer
- **Shinobi Systems**: Development team
- **Community Contributors**: Bug reports and features
- **FFmpeg Project**: Video processing foundation

## 15. Version History

### Current Releases
- **v4.0+**: Modern web interface, improved performance
- **v3.x**: Stable release with plugin system
- **v2.x**: Legacy version (no longer maintained)

### Major Features by Version
- **v4.0**: React-based UI, improved API
- **v3.5**: Plugin marketplace
- **v3.0**: Multi-tenant support
- **v2.5**: Motion detection improvements

## 16. Appendices

### A. Camera Configuration Examples

#### Generic RTSP Camera
```json
{
  "name": "Front Door",
  "type": "rtsp",
  "path": "rtsp://username:password@192.168.1.100:554/stream1",
  "port": 554,
  "host": "192.168.1.100",
  "username": "admin",
  "password": "camera_password"
}
```

#### Hikvision Camera
```json
{
  "name": "Hikvision Cam",
  "type": "rtsp", 
  "path": "rtsp://admin:password@192.168.1.101:554/Streaming/Channels/101",
  "port": 554,
  "host": "192.168.1.101"
}
```

#### Dahua Camera
```json
{
  "name": "Dahua Cam",
  "type": "rtsp",
  "path": "rtsp://admin:password@192.168.1.102:554/cam/realmonitor?channel=1&subtype=0",
  "port": 554,
  "host": "192.168.1.102"
}
```

### B. API Examples

#### Get Monitor List
```bash
curl -X GET "http://localhost:8080/api/monitor/list" \
  -H "Content-Type: application/json" \
  -d '{"ke":"your_api_key","uid":"your_user_id"}'
```

#### Start Recording
```bash
curl -X POST "http://localhost:8080/api/monitor/record/start" \
  -H "Content-Type: application/json" \
  -d '{"ke":"your_api_key","uid":"your_user_id","mid":"monitor_id"}'
```

### C. Performance Monitoring

```bash
#!/bin/bash
# Monitor Shinobi performance

echo "=== CPU Usage ==="
top -bn1 | grep shinobi

echo "=== Memory Usage ==="
ps aux | grep shinobi | awk '{sum+=$6} END {print "Total Memory: " sum/1024 " MB"}'

echo "=== Disk Usage ==="
df -h /opt/shinobi/videos

echo "=== Active Connections ==="
netstat -an | grep :8080 | wc -l

echo "=== Database Size ==="
mysql -u ccio -p ccio -e "SELECT table_schema AS 'Database', 
ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS 'MB' 
FROM information_schema.tables WHERE table_schema='ccio';"
```

---

For more information and updates, visit https://github.com/howtomgr/shinobi
