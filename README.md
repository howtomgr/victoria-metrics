# victoria-metrics Installation Guide

victoria-metrics is a free and open-source fast, cost-effective monitoring solution and time series database. VictoriaMetrics provides long-term storage for Prometheus with improved compression and performance, serving as an alternative to Prometheus, InfluxDB, or TimescaleDB

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
  - CPU: 2+ cores recommended
  - RAM: 1GB minimum (8GB+ for production)
  - Storage: 10GB+ for metrics
  - Network: HTTP for Prometheus API
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8428 (default victoria-metrics port)
  - Port 8089 for InfluxDB protocol
- **Dependencies**:
  - See official documentation for specific requirements
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

# Install victoria-metrics
sudo dnf install -y victoria-metrics

# Enable and start service
sudo systemctl enable --now victoria-metrics

# Configure firewall
sudo firewall-cmd --permanent --add-port=8428/tcp
sudo firewall-cmd --reload

# Verify installation
victoria-metrics --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install victoria-metrics
sudo apt install -y victoria-metrics

# Enable and start service
sudo systemctl enable --now victoria-metrics

# Configure firewall
sudo ufw allow 8428

# Verify installation
victoria-metrics --version
```

### Arch Linux

```bash
# Install victoria-metrics
sudo pacman -S victoria-metrics

# Enable and start service
sudo systemctl enable --now victoria-metrics

# Verify installation
victoria-metrics --version
```

### Alpine Linux

```bash
# Install victoria-metrics
apk add --no-cache victoria-metrics

# Enable and start service
rc-update add victoria-metrics default
rc-service victoria-metrics start

# Verify installation
victoria-metrics --version
```

### openSUSE/SLES

```bash
# Install victoria-metrics
sudo zypper install -y victoria-metrics

# Enable and start service
sudo systemctl enable --now victoria-metrics

# Configure firewall
sudo firewall-cmd --permanent --add-port=8428/tcp
sudo firewall-cmd --reload

# Verify installation
victoria-metrics --version
```

### macOS

```bash
# Using Homebrew
brew install victoria-metrics

# Start service
brew services start victoria-metrics

# Verify installation
victoria-metrics --version
```

### FreeBSD

```bash
# Using pkg
pkg install victoria-metrics

# Enable in rc.conf
echo 'victoria-metrics_enable="YES"' >> /etc/rc.conf

# Start service
service victoria-metrics start

# Verify installation
victoria-metrics --version
```

### Windows

```bash
# Using Chocolatey
choco install victoria-metrics

# Or using Scoop
scoop install victoria-metrics

# Verify installation
victoria-metrics --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/victoria-metrics

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
victoria-metrics --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable victoria-metrics

# Start service
sudo systemctl start victoria-metrics

# Stop service
sudo systemctl stop victoria-metrics

# Restart service
sudo systemctl restart victoria-metrics

# Check status
sudo systemctl status victoria-metrics

# View logs
sudo journalctl -u victoria-metrics -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add victoria-metrics default

# Start service
rc-service victoria-metrics start

# Stop service
rc-service victoria-metrics stop

# Restart service
rc-service victoria-metrics restart

# Check status
rc-service victoria-metrics status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'victoria-metrics_enable="YES"' >> /etc/rc.conf

# Start service
service victoria-metrics start

# Stop service
service victoria-metrics stop

# Restart service
service victoria-metrics restart

# Check status
service victoria-metrics status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start victoria-metrics
brew services stop victoria-metrics
brew services restart victoria-metrics

# Check status
brew services list | grep victoria-metrics
```

### Windows Service Manager

```powershell
# Start service
net start victoria-metrics

# Stop service
net stop victoria-metrics

# Using PowerShell
Start-Service victoria-metrics
Stop-Service victoria-metrics
Restart-Service victoria-metrics

# Check status
Get-Service victoria-metrics
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream victoria-metrics_backend {
    server 127.0.0.1:8428;
}

server {
    listen 80;
    server_name victoria-metrics.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name victoria-metrics.example.com;

    ssl_certificate /etc/ssl/certs/victoria-metrics.example.com.crt;
    ssl_certificate_key /etc/ssl/private/victoria-metrics.example.com.key;

    location / {
        proxy_pass http://victoria-metrics_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName victoria-metrics.example.com
    Redirect permanent / https://victoria-metrics.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName victoria-metrics.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/victoria-metrics.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/victoria-metrics.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8428/
    ProxyPassReverse / http://127.0.0.1:8428/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend victoria-metrics_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/victoria-metrics.pem
    redirect scheme https if !{ ssl_fc }
    default_backend victoria-metrics_backend

backend victoria-metrics_backend
    balance roundrobin
    server victoria-metrics1 127.0.0.1:8428 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R victoria-metrics:victoria-metrics /etc/victoria-metrics
sudo chmod 750 /etc/victoria-metrics

# Configure firewall
sudo firewall-cmd --permanent --add-port=8428/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

See official documentation for database configuration requirements.

## Performance Optimization

### System Tuning

```bash
# Basic system tuning
echo 'net.core.somaxconn = 65535' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65535' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Monitoring

### Basic Monitoring

```bash
# Check service status
sudo systemctl status victoria-metrics

# View logs
sudo journalctl -u victoria-metrics -f

# Monitor resource usage
top -p $(pgrep victoria-metrics)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/victoria-metrics"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/victoria-metrics-backup-$DATE.tar.gz" /etc/victoria-metrics /var/lib/victoria-metrics

echo "Backup completed: $BACKUP_DIR/victoria-metrics-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop victoria-metrics

# Restore from backup
tar -xzf /backup/victoria-metrics/victoria-metrics-backup-*.tar.gz -C /

# Start service
sudo systemctl start victoria-metrics
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u victoria-metrics -n 100
sudo tail -f /var/log/victoria-metrics/victoria-metrics.log

# Check configuration
victoria-metrics --version

# Check permissions
ls -la /etc/victoria-metrics
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8428

# Test connectivity
telnet localhost 8428

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep victoria-metrics)

# Check disk I/O
iotop -p $(pgrep victoria-metrics)

# Check connections
ss -an | grep 8428
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  victoria-metrics:
    image: victoria-metrics:latest
    ports:
      - "8428:8428"
    volumes:
      - ./config:/etc/victoria-metrics
      - ./data:/var/lib/victoria-metrics
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update victoria-metrics

# Debian/Ubuntu
sudo apt update && sudo apt upgrade victoria-metrics

# Arch Linux
sudo pacman -Syu victoria-metrics

# Alpine Linux
apk update && apk upgrade victoria-metrics

# openSUSE
sudo zypper update victoria-metrics

# FreeBSD
pkg update && pkg upgrade victoria-metrics

# Always backup before updates
tar -czf /backup/victoria-metrics-pre-update-$(date +%Y%m%d).tar.gz /etc/victoria-metrics

# Restart after updates
sudo systemctl restart victoria-metrics
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/victoria-metrics

# Clean old logs
find /var/log/victoria-metrics -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/victoria-metrics
```

## Additional Resources

- Official Documentation: https://docs.victoria-metrics.org/
- GitHub Repository: https://github.com/victoria-metrics/victoria-metrics
- Community Forum: https://forum.victoria-metrics.org/
- Best Practices Guide: https://docs.victoria-metrics.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
