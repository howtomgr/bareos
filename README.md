# bareos Installation Guide

bareos is a free and open-source backup archiving recovery. Fork of Bacula, Bareos provides reliable backup and recovery

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
  - CPU: 2+ cores
  - RAM: 2GB minimum
  - Storage: 100GB for backups
  - Network: Network backup
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 9101 (default bareos port)
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

# Install bareos
sudo dnf install -y bareos

# Enable and start service
sudo systemctl enable --now bareos

# Configure firewall
sudo firewall-cmd --permanent --add-port=9101/tcp
sudo firewall-cmd --reload

# Verify installation
bareos --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install bareos
sudo apt install -y bareos

# Enable and start service
sudo systemctl enable --now bareos

# Configure firewall
sudo ufw allow 9101

# Verify installation
bareos --version
```

### Arch Linux

```bash
# Install bareos
sudo pacman -S bareos

# Enable and start service
sudo systemctl enable --now bareos

# Verify installation
bareos --version
```

### Alpine Linux

```bash
# Install bareos
apk add --no-cache bareos

# Enable and start service
rc-update add bareos default
rc-service bareos start

# Verify installation
bareos --version
```

### openSUSE/SLES

```bash
# Install bareos
sudo zypper install -y bareos

# Enable and start service
sudo systemctl enable --now bareos

# Configure firewall
sudo firewall-cmd --permanent --add-port=9101/tcp
sudo firewall-cmd --reload

# Verify installation
bareos --version
```

### macOS

```bash
# Using Homebrew
brew install bareos

# Start service
brew services start bareos

# Verify installation
bareos --version
```

### FreeBSD

```bash
# Using pkg
pkg install bareos

# Enable in rc.conf
echo 'bareos_enable="YES"' >> /etc/rc.conf

# Start service
service bareos start

# Verify installation
bareos --version
```

### Windows

```bash
# Using Chocolatey
choco install bareos

# Or using Scoop
scoop install bareos

# Verify installation
bareos --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/bareos

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
bareos --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable bareos

# Start service
sudo systemctl start bareos

# Stop service
sudo systemctl stop bareos

# Restart service
sudo systemctl restart bareos

# Check status
sudo systemctl status bareos

# View logs
sudo journalctl -u bareos -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add bareos default

# Start service
rc-service bareos start

# Stop service
rc-service bareos stop

# Restart service
rc-service bareos restart

# Check status
rc-service bareos status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'bareos_enable="YES"' >> /etc/rc.conf

# Start service
service bareos start

# Stop service
service bareos stop

# Restart service
service bareos restart

# Check status
service bareos status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start bareos
brew services stop bareos
brew services restart bareos

# Check status
brew services list | grep bareos
```

### Windows Service Manager

```powershell
# Start service
net start bareos

# Stop service
net stop bareos

# Using PowerShell
Start-Service bareos
Stop-Service bareos
Restart-Service bareos

# Check status
Get-Service bareos
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream bareos_backend {
    server 127.0.0.1:9101;
}

server {
    listen 80;
    server_name bareos.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name bareos.example.com;

    ssl_certificate /etc/ssl/certs/bareos.example.com.crt;
    ssl_certificate_key /etc/ssl/private/bareos.example.com.key;

    location / {
        proxy_pass http://bareos_backend;
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
    ServerName bareos.example.com
    Redirect permanent / https://bareos.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName bareos.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/bareos.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/bareos.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:9101/
    ProxyPassReverse / http://127.0.0.1:9101/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend bareos_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/bareos.pem
    redirect scheme https if !{ ssl_fc }
    default_backend bareos_backend

backend bareos_backend
    balance roundrobin
    server bareos1 127.0.0.1:9101 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R bareos:bareos /etc/bareos
sudo chmod 750 /etc/bareos

# Configure firewall
sudo firewall-cmd --permanent --add-port=9101/tcp
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
sudo systemctl status bareos

# View logs
sudo journalctl -u bareos -f

# Monitor resource usage
top -p $(pgrep bareos)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/bareos"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/bareos-backup-$DATE.tar.gz" /etc/bareos /var/lib/bareos

echo "Backup completed: $BACKUP_DIR/bareos-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop bareos

# Restore from backup
tar -xzf /backup/bareos/bareos-backup-*.tar.gz -C /

# Start service
sudo systemctl start bareos
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u bareos -n 100
sudo tail -f /var/log/bareos/bareos.log

# Check configuration
bareos --version

# Check permissions
ls -la /etc/bareos
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 9101

# Test connectivity
telnet localhost 9101

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep bareos)

# Check disk I/O
iotop -p $(pgrep bareos)

# Check connections
ss -an | grep 9101
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  bareos:
    image: bareos:latest
    ports:
      - "9101:9101"
    volumes:
      - ./config:/etc/bareos
      - ./data:/var/lib/bareos
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update bareos

# Debian/Ubuntu
sudo apt update && sudo apt upgrade bareos

# Arch Linux
sudo pacman -Syu bareos

# Alpine Linux
apk update && apk upgrade bareos

# openSUSE
sudo zypper update bareos

# FreeBSD
pkg update && pkg upgrade bareos

# Always backup before updates
tar -czf /backup/bareos-pre-update-$(date +%Y%m%d).tar.gz /etc/bareos

# Restart after updates
sudo systemctl restart bareos
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/bareos

# Clean old logs
find /var/log/bareos -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/bareos
```

## Additional Resources

- Official Documentation: https://docs.bareos.org/
- GitHub Repository: https://github.com/bareos/bareos
- Community Forum: https://forum.bareos.org/
- Best Practices Guide: https://docs.bareos.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
