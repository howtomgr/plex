# plex Installation Guide

plex is a free and open-source media server for streaming. Plex organizes media and streams to devices, though the server is proprietary with some FOSS clients available

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
  - Storage: 100GB+ for media
  - Network: Streaming bandwidth
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 32400 (default plex port)
  - Various streaming ports
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

# Install plex
sudo dnf install -y plex

# Enable and start service
sudo systemctl enable --now plex

# Configure firewall
sudo firewall-cmd --permanent --add-port=32400/tcp
sudo firewall-cmd --reload

# Verify installation
plex --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install plex
sudo apt install -y plex

# Enable and start service
sudo systemctl enable --now plex

# Configure firewall
sudo ufw allow 32400

# Verify installation
plex --version
```

### Arch Linux

```bash
# Install plex
sudo pacman -S plex

# Enable and start service
sudo systemctl enable --now plex

# Verify installation
plex --version
```

### Alpine Linux

```bash
# Install plex
apk add --no-cache plex

# Enable and start service
rc-update add plex default
rc-service plex start

# Verify installation
plex --version
```

### openSUSE/SLES

```bash
# Install plex
sudo zypper install -y plex

# Enable and start service
sudo systemctl enable --now plex

# Configure firewall
sudo firewall-cmd --permanent --add-port=32400/tcp
sudo firewall-cmd --reload

# Verify installation
plex --version
```

### macOS

```bash
# Using Homebrew
brew install plex

# Start service
brew services start plex

# Verify installation
plex --version
```

### FreeBSD

```bash
# Using pkg
pkg install plex

# Enable in rc.conf
echo 'plex_enable="YES"' >> /etc/rc.conf

# Start service
service plex start

# Verify installation
plex --version
```

### Windows

```bash
# Using Chocolatey
choco install plex

# Or using Scoop
scoop install plex

# Verify installation
plex --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/plex

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
plex --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable plex

# Start service
sudo systemctl start plex

# Stop service
sudo systemctl stop plex

# Restart service
sudo systemctl restart plex

# Check status
sudo systemctl status plex

# View logs
sudo journalctl -u plex -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add plex default

# Start service
rc-service plex start

# Stop service
rc-service plex stop

# Restart service
rc-service plex restart

# Check status
rc-service plex status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'plex_enable="YES"' >> /etc/rc.conf

# Start service
service plex start

# Stop service
service plex stop

# Restart service
service plex restart

# Check status
service plex status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start plex
brew services stop plex
brew services restart plex

# Check status
brew services list | grep plex
```

### Windows Service Manager

```powershell
# Start service
net start plex

# Stop service
net stop plex

# Using PowerShell
Start-Service plex
Stop-Service plex
Restart-Service plex

# Check status
Get-Service plex
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream plex_backend {
    server 127.0.0.1:32400;
}

server {
    listen 80;
    server_name plex.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name plex.example.com;

    ssl_certificate /etc/ssl/certs/plex.example.com.crt;
    ssl_certificate_key /etc/ssl/private/plex.example.com.key;

    location / {
        proxy_pass http://plex_backend;
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
    ServerName plex.example.com
    Redirect permanent / https://plex.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName plex.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/plex.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/plex.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:32400/
    ProxyPassReverse / http://127.0.0.1:32400/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend plex_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/plex.pem
    redirect scheme https if !{ ssl_fc }
    default_backend plex_backend

backend plex_backend
    balance roundrobin
    server plex1 127.0.0.1:32400 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R plex:plex /etc/plex
sudo chmod 750 /etc/plex

# Configure firewall
sudo firewall-cmd --permanent --add-port=32400/tcp
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
sudo systemctl status plex

# View logs
sudo journalctl -u plex -f

# Monitor resource usage
top -p $(pgrep plex)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/plex"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/plex-backup-$DATE.tar.gz" /etc/plex /var/lib/plex

echo "Backup completed: $BACKUP_DIR/plex-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop plex

# Restore from backup
tar -xzf /backup/plex/plex-backup-*.tar.gz -C /

# Start service
sudo systemctl start plex
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u plex -n 100
sudo tail -f /var/log/plex/plex.log

# Check configuration
plex --version

# Check permissions
ls -la /etc/plex
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 32400

# Test connectivity
telnet localhost 32400

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep plex)

# Check disk I/O
iotop -p $(pgrep plex)

# Check connections
ss -an | grep 32400
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  plex:
    image: plex:latest
    ports:
      - "32400:32400"
    volumes:
      - ./config:/etc/plex
      - ./data:/var/lib/plex
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update plex

# Debian/Ubuntu
sudo apt update && sudo apt upgrade plex

# Arch Linux
sudo pacman -Syu plex

# Alpine Linux
apk update && apk upgrade plex

# openSUSE
sudo zypper update plex

# FreeBSD
pkg update && pkg upgrade plex

# Always backup before updates
tar -czf /backup/plex-pre-update-$(date +%Y%m%d).tar.gz /etc/plex

# Restart after updates
sudo systemctl restart plex
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/plex

# Clean old logs
find /var/log/plex -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/plex
```

## Additional Resources

- Official Documentation: https://docs.plex.org/
- GitHub Repository: https://github.com/plex/plex
- Community Forum: https://forum.plex.org/
- Best Practices Guide: https://docs.plex.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
