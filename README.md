# ejabberd Installation Guide

ejabberd is a free and open-source XMPP Server. Robust, scalable and extensible XMPP server

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
  - Network: 5222/5269 ports
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 5222/5269 (default ejabberd port)
- **Dependencies**:
  - erlang, openssl
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

# Install ejabberd
sudo dnf install -y ejabberd erlang, openssl

# Enable and start service
sudo systemctl enable --now ejabberd

# Configure firewall
sudo firewall-cmd --permanent --add-service=ejabberd
sudo firewall-cmd --reload

# Verify installation
ejabberd --version || systemctl status ejabberd
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install ejabberd
sudo apt install -y ejabberd erlang, openssl

# Enable and start service
sudo systemctl enable --now ejabberd

# Configure firewall
sudo ufw allow 5222/5269

# Verify installation
ejabberd --version || systemctl status ejabberd
```

### Arch Linux

```bash
# Install ejabberd
sudo pacman -S ejabberd

# Enable and start service
sudo systemctl enable --now ejabberd

# Verify installation
ejabberd --version || systemctl status ejabberd
```

### Alpine Linux

```bash
# Install ejabberd
apk add --no-cache ejabberd

# Enable and start service
rc-update add ejabberd default
rc-service ejabberd start

# Verify installation
ejabberd --version || rc-service ejabberd status
```

### openSUSE/SLES

```bash
# Install ejabberd
sudo zypper install -y ejabberd erlang, openssl

# Enable and start service
sudo systemctl enable --now ejabberd

# Configure firewall
sudo firewall-cmd --permanent --add-service=ejabberd
sudo firewall-cmd --reload

# Verify installation
ejabberd --version || systemctl status ejabberd
```

### macOS

```bash
# Using Homebrew
brew install ejabberd

# Start service
brew services start ejabberd

# Verify installation
ejabberd --version
```

### FreeBSD

```bash
# Using pkg
pkg install ejabberd

# Enable in rc.conf
echo 'ejabberd_enable="YES"' >> /etc/rc.conf

# Start service
service ejabberd start

# Verify installation
ejabberd --version || service ejabberd status
```

### Windows

```powershell
# Using Chocolatey
choco install ejabberd

# Or using Scoop
scoop install ejabberd

# Verify installation
ejabberd --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory if needed
sudo mkdir -p /etc/ejabberd

# Set up basic configuration
sudo tee /etc/ejabberd/ejabberd.conf << 'EOF'
# ejabberd Configuration
max_connections: 10000
EOF

# Test configuration
sudo ejabberd -t || sudo ejabberd configtest

# Reload service
sudo systemctl reload ejabberd
```

### Security Hardening

```bash
# Set appropriate permissions
sudo chown -R ejabberd:ejabberd /etc/ejabberd
sudo chmod 750 /etc/ejabberd

# Enable security features
# See security section for detailed hardening steps
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable ejabberd

# Start service
sudo systemctl start ejabberd

# Stop service
sudo systemctl stop ejabberd

# Restart service
sudo systemctl restart ejabberd

# Reload configuration
sudo systemctl reload ejabberd

# Check status
sudo systemctl status ejabberd

# View logs
sudo journalctl -u ejabberd -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add ejabberd default

# Start service
rc-service ejabberd start

# Stop service
rc-service ejabberd stop

# Restart service
rc-service ejabberd restart

# Check status
rc-service ejabberd status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'ejabberd_enable="YES"' >> /etc/rc.conf

# Start service
service ejabberd start

# Stop service
service ejabberd stop

# Restart service
service ejabberd restart

# Check status
service ejabberd status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start ejabberd
brew services stop ejabberd
brew services restart ejabberd

# Check status
brew services list | grep ejabberd
```

### Windows Service Manager

```powershell
# Start service
net start ejabberd

# Stop service
net stop ejabberd

# Using PowerShell
Start-Service ejabberd
Stop-Service ejabberd
Restart-Service ejabberd

# Check status
Get-Service ejabberd
```

## Advanced Configuration

### Performance Optimization

```bash
# Configure performance settings
cat >> /etc/ejabberd/ejabberd.conf << 'EOF'
max_connections: 10000
EOF

# Apply system tuning
sudo sysctl -w net.core.somaxconn=65535
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65535

# Restart service
sudo systemctl restart ejabberd
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
upstream ejabberd_backend {
    server 127.0.0.1:5222/5269;
    server 127.0.0.1:{default_port}1 backup;
}

server {
    listen 80;
    server_name ejabberd.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name ejabberd.example.com;

    ssl_certificate /etc/ssl/certs/ejabberd.example.com.crt;
    ssl_certificate_key /etc/ssl/private/ejabberd.example.com.key;

    location / {
        proxy_pass http://ejabberd_backend;
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
    ServerName ejabberd.example.com
    Redirect permanent / https://ejabberd.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName ejabberd.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/ejabberd.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/ejabberd.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:5222/5269/
    ProxyPassReverse / http://127.0.0.1:5222/5269/
    
    # WebSocket support (if needed)
    RewriteEngine on
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^/?(.*) "ws://127.0.0.1:5222/5269/$1" [P,L]
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend ejabberd_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/ejabberd.pem
    redirect scheme https if !{ ssl_fc }
    default_backend ejabberd_backend

backend ejabberd_backend
    balance roundrobin
    option httpchk GET /health
    server ejabberd1 127.0.0.1:5222/5269 check
    server ejabberd2 127.0.0.1:{default_port}1 check backup
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R ejabberd:ejabberd /etc/ejabberd
sudo chmod 750 /etc/ejabberd

# Configure firewall
sudo firewall-cmd --permanent --add-service=ejabberd
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on

# Configure fail2ban
sudo tee /etc/fail2ban/jail.d/ejabberd.conf << 'EOF'
[ejabberd]
enabled = true
port = 5222/5269
filter = ejabberd
logpath = /var/log/ejabberd/*.log
maxretry = 5
bantime = 3600
EOF
```

### SSL/TLS Configuration

```bash
# Generate SSL certificates
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/ejabberd.key \
    -out /etc/ssl/certs/ejabberd.crt

# Configure SSL in ejabberd
# See official documentation for SSL configuration
```

## Database Setup

### PostgreSQL Backend (if applicable)

```bash
# Create database and user
sudo -u postgres psql << EOF
CREATE DATABASE ejabberd_db;
CREATE USER ejabberd_user WITH ENCRYPTED PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE ejabberd_db TO ejabberd_user;
EOF

# Configure ejabberd to use PostgreSQL
# See official documentation for database configuration
```

### MySQL/MariaDB Backend (if applicable)

```bash
# Create database and user
sudo mysql << EOF
CREATE DATABASE ejabberd_db;
CREATE USER 'ejabberd_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON ejabberd_db.* TO 'ejabberd_user'@'localhost';
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

# ejabberd specific tuning
max_connections: 10000
```

### Resource Limits

```bash
# Configure system limits
sudo tee -a /etc/security/limits.conf << EOF
ejabberd soft nofile 65535
ejabberd hard nofile 65535
ejabberd soft nproc 32768
ejabberd hard nproc 32768
EOF
```

## Monitoring

### Prometheus Integration

```yaml
# prometheus.yml configuration
scrape_configs:
  - job_name: 'ejabberd'
    static_configs:
      - targets: ['localhost:5222/5269']
    metrics_path: '/metrics'
```

### Health Checks

```bash
# Basic health check script
#!/bin/bash
if systemctl is-active --quiet ejabberd; then
    echo "ejabberd is running"
    exit 0
else
    echo "ejabberd is not running"
    exit 1
fi
```

### Log Monitoring

```bash
# Configure log rotation
sudo tee /etc/logrotate.d/ejabberd << 'EOF'
/var/log/ejabberd/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 ejabberd ejabberd
    postrotate
        systemctl reload ejabberd > /dev/null 2>&1 || true
    endscript
}
EOF
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# ejabberd backup script
BACKUP_DIR="/backup/ejabberd"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Stop service (if required)
systemctl stop ejabberd

# Backup configuration
tar -czf "$BACKUP_DIR/ejabberd-config-$DATE.tar.gz" /etc/ejabberd

# Backup data (adjust paths as needed)
tar -czf "$BACKUP_DIR/ejabberd-data-$DATE.tar.gz" /var/lib/ejabberd

# Start service
systemctl start ejabberd

# Clean old backups (keep 30 days)
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete

echo "Backup completed: $BACKUP_DIR"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop ejabberd

# Restore configuration
sudo tar -xzf /backup/ejabberd/ejabberd-config-*.tar.gz -C /

# Restore data
sudo tar -xzf /backup/ejabberd/ejabberd-data-*.tar.gz -C /

# Set permissions
sudo chown -R ejabberd:ejabberd /etc/ejabberd
sudo chown -R ejabberd:ejabberd /var/lib/ejabberd

# Start service
sudo systemctl start ejabberd
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u ejabberd -n 100
sudo tail -f /var/log/ejabberd/*.log

# Check configuration
sudo ejabberd -t || sudo ejabberd configtest

# Check permissions
ls -la /etc/ejabberd
ls -la /var/lib/ejabberd
```

2. **Connection refused**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 5222/5269
sudo netstat -tlnp | grep 5222/5269

# Check firewall
sudo firewall-cmd --list-all
sudo iptables -L -n

# Test connection
telnet localhost 5222/5269
nc -zv localhost 5222/5269
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep beam.smp)
htop -p $(pgrep beam.smp)

# Check connections
ss -ant | grep :5222/5269 | wc -l

# Monitor I/O
iotop -p $(pgrep beam.smp)
```

### Debug Mode

```bash
# Run in debug mode
sudo ejabberd -d
# or
sudo ejabberd debug

# Increase log verbosity
# Edit configuration to enable debug logging
```

## Integration Examples

### Docker Compose

```yaml
version: '3.8'
services:
  ejabberd:
    image: ejabberd:latest
    container_name: ejabberd
    ports:
      - "5222/5269:5222/5269"
    volumes:
      - ./config:/etc/ejabberd
      - ./data:/var/lib/ejabberd
    environment:
      - ejabberd_CONFIG=/etc/ejabberd/ejabberd.conf
    restart: unless-stopped
    networks:
      - ejabberd_net

networks:
  ejabberd_net:
    driver: bridge
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ejabberd
spec:
  replicas: 3
  selector:
    matchLabels:
      app: ejabberd
  template:
    metadata:
      labels:
        app: ejabberd
    spec:
      containers:
      - name: ejabberd
        image: ejabberd:latest
        ports:
        - containerPort: 5222/5269
        volumeMounts:
        - name: config
          mountPath: /etc/ejabberd
      volumes:
      - name: config
        configMap:
          name: ejabberd-config
---
apiVersion: v1
kind: Service
metadata:
  name: ejabberd
spec:
  selector:
    app: ejabberd
  ports:
  - port: 5222/5269
    targetPort: 5222/5269
  type: LoadBalancer
```

### Ansible Playbook

```yaml
---
- name: Install and configure ejabberd
  hosts: all
  become: yes
  tasks:
    - name: Install ejabberd
      package:
        name: ejabberd
        state: present
    
    - name: Configure ejabberd
      template:
        src: ejabberd.conf.j2
        dest: /etc/ejabberd/ejabberd.conf
        owner: ejabberd
        group: ejabberd
        mode: '0640'
      notify: restart ejabberd
    
    - name: Start and enable ejabberd
      systemd:
        name: ejabberd
        state: started
        enabled: yes
  
  handlers:
    - name: restart ejabberd
      systemd:
        name: ejabberd
        state: restarted
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update ejabberd

# Debian/Ubuntu
sudo apt update && sudo apt upgrade ejabberd

# Arch Linux
sudo pacman -Syu ejabberd

# Alpine Linux
apk update && apk upgrade ejabberd

# openSUSE
sudo zypper update ejabberd

# FreeBSD
pkg update && pkg upgrade ejabberd

# Always backup before updates
tar -czf /backup/ejabberd-pre-update-$(date +%Y%m%d).tar.gz /etc/ejabberd

# Restart after updates
sudo systemctl restart ejabberd
```

### Regular Maintenance Tasks

```bash
# Clean logs
find /var/log/ejabberd -name "*.log" -mtime +30 -delete

# Verify integrity
sudo ejabberd --verify || sudo ejabberd check

# Update databases (if applicable)
sudo ejabberd-update-db

# Optimize performance
sudo ejabberd-optimize

# Check for security updates
sudo ejabberd --security-check
```

## Additional Resources

- Official Documentation: https://docs.ejabberd.org/
- GitHub Repository: https://github.com/ejabberd/ejabberd
- Community Forum: https://forum.ejabberd.org/
- Wiki: https://wiki.ejabberd.org/
- Comparison vs Prosody, Openfire, Tigase, MongooseIM: https://docs.ejabberd.org/comparison

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
