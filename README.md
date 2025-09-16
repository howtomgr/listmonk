# Listmonk Installation Guide

Listmonk is a free and open-source Newsletter. Self-hosted newsletter and mailing list manager

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

- **Hardware Requirements**:
  - CPU: 2 cores minimum (4+ cores recommended)
  - RAM: 2GB minimum (4GB+ recommended)
  - Storage: 1GB for installation
  - Network: 9000 ports
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 9000 (default listmonk port)
- **Dependencies**:
  - postgresql
- **System Access**: root or sudo privileges required


## 2. Supported Operating Systems

This guide supports installation on:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux (rolling release)
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later) 
- FreeBSD 13+
- Windows 10/11/Server 2019+ (where applicable)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Install EPEL repository if needed
sudo dnf install -y epel-release

# Install listmonk
sudo dnf install -y listmonk postgresql

# Enable and start service
sudo systemctl enable --now listmonk

# Configure firewall
sudo firewall-cmd --permanent --add-service=listmonk
sudo firewall-cmd --reload

# Verify installation
listmonk --version || systemctl status listmonk
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install listmonk
sudo apt install -y listmonk postgresql

# Enable and start service
sudo systemctl enable --now listmonk

# Configure firewall
sudo ufw allow 9000

# Verify installation
listmonk --version || systemctl status listmonk
```

### Arch Linux

```bash
# Install listmonk
sudo pacman -S listmonk

# Enable and start service
sudo systemctl enable --now listmonk

# Verify installation
listmonk --version || systemctl status listmonk
```

### Alpine Linux

```bash
# Install listmonk
apk add --no-cache listmonk

# Enable and start service
rc-update add listmonk default
rc-service listmonk start

# Verify installation
listmonk --version || rc-service listmonk status
```

### openSUSE/SLES

```bash
# Install listmonk
sudo zypper install -y listmonk postgresql

# Enable and start service
sudo systemctl enable --now listmonk

# Configure firewall
sudo firewall-cmd --permanent --add-service=listmonk
sudo firewall-cmd --reload

# Verify installation
listmonk --version || systemctl status listmonk
```

### macOS

```bash
# Using Homebrew
brew install listmonk

# Start service
brew services start listmonk

# Verify installation
listmonk --version
```

### FreeBSD

```bash
# Using pkg
pkg install listmonk

# Enable in rc.conf
echo 'listmonk_enable="YES"' >> /etc/rc.conf

# Start service
service listmonk start

# Verify installation
listmonk --version || service listmonk status
```

### Windows

```powershell
# Using Chocolatey
choco install listmonk

# Or using Scoop
scoop install listmonk

# Verify installation
listmonk --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory if needed
sudo mkdir -p /etc/listmonk

# Set up basic configuration
sudo tee /etc/listmonk/listmonk.conf << 'EOF'
# Listmonk Configuration
concurrency = 100
EOF

# Test configuration
sudo listmonk -t || sudo listmonk configtest

# Reload service
sudo systemctl reload listmonk
```

### Security Hardening

```bash
# Set appropriate permissions
sudo chown -R listmonk:listmonk /etc/listmonk
sudo chmod 750 /etc/listmonk

# Enable security features
# See security section for detailed hardening steps
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable listmonk

# Start service
sudo systemctl start listmonk

# Stop service
sudo systemctl stop listmonk

# Restart service
sudo systemctl restart listmonk

# Reload configuration
sudo systemctl reload listmonk

# Check status
sudo systemctl status listmonk

# View logs
sudo journalctl -u listmonk -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add listmonk default

# Start service
rc-service listmonk start

# Stop service
rc-service listmonk stop

# Restart service
rc-service listmonk restart

# Check status
rc-service listmonk status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'listmonk_enable="YES"' >> /etc/rc.conf

# Start service
service listmonk start

# Stop service
service listmonk stop

# Restart service
service listmonk restart

# Check status
service listmonk status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start listmonk
brew services stop listmonk
brew services restart listmonk

# Check status
brew services list | grep listmonk
```

### Windows Service Manager

```powershell
# Start service
net start listmonk

# Stop service
net stop listmonk

# Using PowerShell
Start-Service listmonk
Stop-Service listmonk
Restart-Service listmonk

# Check status
Get-Service listmonk
```

## Advanced Configuration

### Performance Optimization

```bash
# Configure performance settings
cat >> /etc/listmonk/listmonk.conf << 'EOF'
concurrency = 100
EOF

# Apply system tuning
sudo sysctl -w net.core.somaxconn=65535
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65535

# Restart service
sudo systemctl restart listmonk
```

### Clustering and High Availability

```bash
# Configure clustering (if supported)
# See official documentation for cluster setup

# Basic load balancing setup example
# Configure multiple instances on different ports
```

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream listmonk_backend {
    server 127.0.0.1:9000;
    server 127.0.0.1:{default_port}1 backup;
}

server {
    listen 80;
    server_name listmonk.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name listmonk.example.com;

    ssl_certificate /etc/ssl/certs/listmonk.example.com.crt;
    ssl_certificate_key /etc/ssl/private/listmonk.example.com.key;

    location / {
        proxy_pass http://listmonk_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support (if needed)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName listmonk.example.com
    Redirect permanent / https://listmonk.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName listmonk.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/listmonk.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/listmonk.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:9000/
    ProxyPassReverse / http://127.0.0.1:9000/
    
    # WebSocket support (if needed)
    RewriteEngine on
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^/?(.*) "ws://127.0.0.1:9000/$1" [P,L]
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend listmonk_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/listmonk.pem
    redirect scheme https if !{ ssl_fc }
    default_backend listmonk_backend

backend listmonk_backend
    balance roundrobin
    option httpchk GET /health
    server listmonk1 127.0.0.1:9000 check
    server listmonk2 127.0.0.1:{default_port}1 check backup
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R listmonk:listmonk /etc/listmonk
sudo chmod 750 /etc/listmonk

# Configure firewall
sudo firewall-cmd --permanent --add-service=listmonk
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on

# Configure fail2ban
sudo tee /etc/fail2ban/jail.d/listmonk.conf << 'EOF'
[listmonk]
enabled = true
port = 9000
filter = listmonk
logpath = /var/log/listmonk/*.log
maxretry = 5
bantime = 3600
EOF
```

### SSL/TLS Configuration

```bash
# Generate SSL certificates
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/listmonk.key \
    -out /etc/ssl/certs/listmonk.crt

# Configure SSL in listmonk
# See official documentation for SSL configuration
```

## Database Setup

### PostgreSQL Backend (if applicable)

```bash
# Create database and user
sudo -u postgres psql << EOF
CREATE DATABASE listmonk_db;
CREATE USER listmonk_user WITH ENCRYPTED PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE listmonk_db TO listmonk_user;
EOF

# Configure listmonk to use PostgreSQL
# See official documentation for database configuration
```

### MySQL/MariaDB Backend (if applicable)

```bash
# Create database and user
sudo mysql << EOF
CREATE DATABASE listmonk_db;
CREATE USER 'listmonk_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON listmonk_db.* TO 'listmonk_user'@'localhost';
FLUSH PRIVILEGES;
EOF
```

## Performance Optimization

### System Tuning

```bash
# Kernel parameters
sudo tee -a /etc/sysctl.conf << EOF
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.ip_local_port_range = 1024 65535
net.core.netdev_max_backlog = 5000
vm.swappiness = 10
EOF

sudo sysctl -p

# Listmonk specific tuning
concurrency = 100
```

### Resource Limits

```bash
# Configure system limits
sudo tee -a /etc/security/limits.conf << EOF
listmonk soft nofile 65535
listmonk hard nofile 65535
listmonk soft nproc 32768
listmonk hard nproc 32768
EOF
```

## Monitoring

### Prometheus Integration

```yaml
# prometheus.yml configuration
scrape_configs:
  - job_name: 'listmonk'
    static_configs:
      - targets: ['localhost:9000']
    metrics_path: '/metrics'
```

### Health Checks

```bash
# Basic health check script
#!/bin/bash
if systemctl is-active --quiet listmonk; then
    echo "Listmonk is running"
    exit 0
else
    echo "Listmonk is not running"
    exit 1
fi
```

### Log Monitoring

```bash
# Configure log rotation
sudo tee /etc/logrotate.d/listmonk << 'EOF'
/var/log/listmonk/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 listmonk listmonk
    postrotate
        systemctl reload listmonk > /dev/null 2>&1 || true
    endscript
}
EOF
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Listmonk backup script
BACKUP_DIR="/backup/listmonk"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Stop service (if required)
systemctl stop listmonk

# Backup configuration
tar -czf "$BACKUP_DIR/listmonk-config-$DATE.tar.gz" /etc/listmonk

# Backup data (adjust paths as needed)
tar -czf "$BACKUP_DIR/listmonk-data-$DATE.tar.gz" /var/lib/listmonk

# Start service
systemctl start listmonk

# Clean old backups (keep 30 days)
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete

echo "Backup completed: $BACKUP_DIR"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop listmonk

# Restore configuration
sudo tar -xzf /backup/listmonk/listmonk-config-*.tar.gz -C /

# Restore data
sudo tar -xzf /backup/listmonk/listmonk-data-*.tar.gz -C /

# Set permissions
sudo chown -R listmonk:listmonk /etc/listmonk
sudo chown -R listmonk:listmonk /var/lib/listmonk

# Start service
sudo systemctl start listmonk
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u listmonk -n 100
sudo tail -f /var/log/listmonk/*.log

# Check configuration
sudo listmonk -t || sudo listmonk configtest

# Check permissions
ls -la /etc/listmonk
ls -la /var/lib/listmonk
```

2. **Connection refused**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 9000
sudo netstat -tlnp | grep 9000

# Check firewall
sudo firewall-cmd --list-all
sudo iptables -L -n

# Test connection
telnet localhost 9000
nc -zv localhost 9000
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep listmonk)
htop -p $(pgrep listmonk)

# Check connections
ss -ant | grep :9000 | wc -l

# Monitor I/O
iotop -p $(pgrep listmonk)
```

### Debug Mode

```bash
# Run in debug mode
sudo listmonk -d
# or
sudo listmonk debug

# Increase log verbosity
# Edit configuration to enable debug logging
```

## Integration Examples

### Docker Compose

```yaml
version: '3.8'
services:
  listmonk:
    image: listmonk:latest
    container_name: listmonk
    ports:
      - "9000:9000"
    volumes:
      - ./config:/etc/listmonk
      - ./data:/var/lib/listmonk
    environment:
      - listmonk_CONFIG=/etc/listmonk/listmonk.conf
    restart: unless-stopped
    networks:
      - listmonk_net

networks:
  listmonk_net:
    driver: bridge
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: listmonk
spec:
  replicas: 3
  selector:
    matchLabels:
      app: listmonk
  template:
    metadata:
      labels:
        app: listmonk
    spec:
      containers:
      - name: listmonk
        image: listmonk:latest
        ports:
        - containerPort: 9000
        volumeMounts:
        - name: config
          mountPath: /etc/listmonk
      volumes:
      - name: config
        configMap:
          name: listmonk-config
---
apiVersion: v1
kind: Service
metadata:
  name: listmonk
spec:
  selector:
    app: listmonk
  ports:
  - port: 9000
    targetPort: 9000
  type: LoadBalancer
```

### Ansible Playbook

```yaml
---
- name: Install and configure Listmonk
  hosts: all
  become: yes
  tasks:
    - name: Install listmonk
      package:
        name: listmonk
        state: present
    
    - name: Configure listmonk
      template:
        src: listmonk.conf.j2
        dest: /etc/listmonk/listmonk.conf
        owner: listmonk
        group: listmonk
        mode: '0640'
      notify: restart listmonk
    
    - name: Start and enable listmonk
      systemd:
        name: listmonk
        state: started
        enabled: yes
  
  handlers:
    - name: restart listmonk
      systemd:
        name: listmonk
        state: restarted
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update listmonk

# Debian/Ubuntu
sudo apt update && sudo apt upgrade listmonk

# Arch Linux
sudo pacman -Syu listmonk

# Alpine Linux
apk update && apk upgrade listmonk

# openSUSE
sudo zypper update listmonk

# FreeBSD
pkg update && pkg upgrade listmonk

# Always backup before updates
tar -czf /backup/listmonk-pre-update-$(date +%Y%m%d).tar.gz /etc/listmonk

# Restart after updates
sudo systemctl restart listmonk
```

### Regular Maintenance Tasks

```bash
# Clean logs
find /var/log/listmonk -name "*.log" -mtime +30 -delete

# Verify integrity
sudo listmonk --verify || sudo listmonk check

# Update databases (if applicable)
sudo listmonk-update-db

# Optimize performance
sudo listmonk-optimize

# Check for security updates
sudo listmonk --security-check
```

## Additional Resources

- Official Documentation: https://docs.listmonk.org/
- GitHub Repository: https://github.com/listmonk/listmonk
- Community Forum: https://forum.listmonk.org/
- Wiki: https://wiki.listmonk.org/
- Comparison vs Mailman, Sympa, phpList, Sendy: https://docs.listmonk.org/comparison

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
