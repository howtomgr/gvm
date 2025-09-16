# gvm Installation Guide

gvm is a free and open-source vulnerability scanning. GVM provides full-featured vulnerability scanning and management

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
  - CPU: 4+ cores
  - RAM: 8GB minimum
  - Storage: 50GB for feeds
  - Network: HTTPS/GMP
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 9392 (default gvm port)
  - Various service ports
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

# Install gvm
sudo dnf install -y gvm

# Enable and start service
sudo systemctl enable --now gvm

# Configure firewall
sudo firewall-cmd --permanent --add-port=9392/tcp
sudo firewall-cmd --reload

# Verify installation
gvm --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install gvm
sudo apt install -y gvm

# Enable and start service
sudo systemctl enable --now gvm

# Configure firewall
sudo ufw allow 9392

# Verify installation
gvm --version
```

### Arch Linux

```bash
# Install gvm
sudo pacman -S gvm

# Enable and start service
sudo systemctl enable --now gvm

# Verify installation
gvm --version
```

### Alpine Linux

```bash
# Install gvm
apk add --no-cache gvm

# Enable and start service
rc-update add gvm default
rc-service gvm start

# Verify installation
gvm --version
```

### openSUSE/SLES

```bash
# Install gvm
sudo zypper install -y gvm

# Enable and start service
sudo systemctl enable --now gvm

# Configure firewall
sudo firewall-cmd --permanent --add-port=9392/tcp
sudo firewall-cmd --reload

# Verify installation
gvm --version
```

### macOS

```bash
# Using Homebrew
brew install gvm

# Start service
brew services start gvm

# Verify installation
gvm --version
```

### FreeBSD

```bash
# Using pkg
pkg install gvm

# Enable in rc.conf
echo 'gvm_enable="YES"' >> /etc/rc.conf

# Start service
service gvm start

# Verify installation
gvm --version
```

### Windows

```bash
# Using Chocolatey
choco install gvm

# Or using Scoop
scoop install gvm

# Verify installation
gvm --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/gvm

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
gvm --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable gvm

# Start service
sudo systemctl start gvm

# Stop service
sudo systemctl stop gvm

# Restart service
sudo systemctl restart gvm

# Check status
sudo systemctl status gvm

# View logs
sudo journalctl -u gvm -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add gvm default

# Start service
rc-service gvm start

# Stop service
rc-service gvm stop

# Restart service
rc-service gvm restart

# Check status
rc-service gvm status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'gvm_enable="YES"' >> /etc/rc.conf

# Start service
service gvm start

# Stop service
service gvm stop

# Restart service
service gvm restart

# Check status
service gvm status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start gvm
brew services stop gvm
brew services restart gvm

# Check status
brew services list | grep gvm
```

### Windows Service Manager

```powershell
# Start service
net start gvm

# Stop service
net stop gvm

# Using PowerShell
Start-Service gvm
Stop-Service gvm
Restart-Service gvm

# Check status
Get-Service gvm
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream gvm_backend {
    server 127.0.0.1:9392;
}

server {
    listen 80;
    server_name gvm.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name gvm.example.com;

    ssl_certificate /etc/ssl/certs/gvm.example.com.crt;
    ssl_certificate_key /etc/ssl/private/gvm.example.com.key;

    location / {
        proxy_pass http://gvm_backend;
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
    ServerName gvm.example.com
    Redirect permanent / https://gvm.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName gvm.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/gvm.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/gvm.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:9392/
    ProxyPassReverse / http://127.0.0.1:9392/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend gvm_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/gvm.pem
    redirect scheme https if !{ ssl_fc }
    default_backend gvm_backend

backend gvm_backend
    balance roundrobin
    server gvm1 127.0.0.1:9392 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R gvm:gvm /etc/gvm
sudo chmod 750 /etc/gvm

# Configure firewall
sudo firewall-cmd --permanent --add-port=9392/tcp
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
sudo systemctl status gvm

# View logs
sudo journalctl -u gvm -f

# Monitor resource usage
top -p $(pgrep gvm)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/gvm"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/gvm-backup-$DATE.tar.gz" /etc/gvm /var/lib/gvm

echo "Backup completed: $BACKUP_DIR/gvm-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop gvm

# Restore from backup
tar -xzf /backup/gvm/gvm-backup-*.tar.gz -C /

# Start service
sudo systemctl start gvm
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u gvm -n 100
sudo tail -f /var/log/gvm/gvm.log

# Check configuration
gvm --version

# Check permissions
ls -la /etc/gvm
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 9392

# Test connectivity
telnet localhost 9392

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep gvm)

# Check disk I/O
iotop -p $(pgrep gvm)

# Check connections
ss -an | grep 9392
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  gvm:
    image: gvm:latest
    ports:
      - "9392:9392"
    volumes:
      - ./config:/etc/gvm
      - ./data:/var/lib/gvm
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update gvm

# Debian/Ubuntu
sudo apt update && sudo apt upgrade gvm

# Arch Linux
sudo pacman -Syu gvm

# Alpine Linux
apk update && apk upgrade gvm

# openSUSE
sudo zypper update gvm

# FreeBSD
pkg update && pkg upgrade gvm

# Always backup before updates
tar -czf /backup/gvm-pre-update-$(date +%Y%m%d).tar.gz /etc/gvm

# Restart after updates
sudo systemctl restart gvm
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/gvm

# Clean old logs
find /var/log/gvm -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/gvm
```

## Additional Resources

- Official Documentation: https://docs.gvm.org/
- GitHub Repository: https://github.com/gvm/gvm
- Community Forum: https://forum.gvm.org/
- Best Practices Guide: https://docs.gvm.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
