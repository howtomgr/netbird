# netbird Installation Guide

netbird is a free and open-source WireGuard-based mesh VPN. NetBird creates secure private networks with SSO support, serving as an open-source alternative to Tailscale with self-hosting

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
  - CPU: 1 core minimum
  - RAM: 256MB minimum
  - Storage: 100MB for installation
  - Network: STUN/TURN servers
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 33073 (default netbird port)
  - Management on 33073
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

# Install netbird
sudo dnf install -y netbird

# Enable and start service
sudo systemctl enable --now netbird

# Configure firewall
sudo firewall-cmd --permanent --add-port=33073/tcp
sudo firewall-cmd --reload

# Verify installation
netbird version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install netbird
sudo apt install -y netbird

# Enable and start service
sudo systemctl enable --now netbird

# Configure firewall
sudo ufw allow 33073

# Verify installation
netbird version
```

### Arch Linux

```bash
# Install netbird
sudo pacman -S netbird

# Enable and start service
sudo systemctl enable --now netbird

# Verify installation
netbird version
```

### Alpine Linux

```bash
# Install netbird
apk add --no-cache netbird

# Enable and start service
rc-update add netbird default
rc-service netbird start

# Verify installation
netbird version
```

### openSUSE/SLES

```bash
# Install netbird
sudo zypper install -y netbird

# Enable and start service
sudo systemctl enable --now netbird

# Configure firewall
sudo firewall-cmd --permanent --add-port=33073/tcp
sudo firewall-cmd --reload

# Verify installation
netbird version
```

### macOS

```bash
# Using Homebrew
brew install netbird

# Start service
brew services start netbird

# Verify installation
netbird version
```

### FreeBSD

```bash
# Using pkg
pkg install netbird

# Enable in rc.conf
echo 'netbird_enable="YES"' >> /etc/rc.conf

# Start service
service netbird start

# Verify installation
netbird version
```

### Windows

```bash
# Using Chocolatey
choco install netbird

# Or using Scoop
scoop install netbird

# Verify installation
netbird version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/netbird

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
netbird version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable netbird

# Start service
sudo systemctl start netbird

# Stop service
sudo systemctl stop netbird

# Restart service
sudo systemctl restart netbird

# Check status
sudo systemctl status netbird

# View logs
sudo journalctl -u netbird -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add netbird default

# Start service
rc-service netbird start

# Stop service
rc-service netbird stop

# Restart service
rc-service netbird restart

# Check status
rc-service netbird status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'netbird_enable="YES"' >> /etc/rc.conf

# Start service
service netbird start

# Stop service
service netbird stop

# Restart service
service netbird restart

# Check status
service netbird status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start netbird
brew services stop netbird
brew services restart netbird

# Check status
brew services list | grep netbird
```

### Windows Service Manager

```powershell
# Start service
net start netbird

# Stop service
net stop netbird

# Using PowerShell
Start-Service netbird
Stop-Service netbird
Restart-Service netbird

# Check status
Get-Service netbird
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream netbird_backend {
    server 127.0.0.1:33073;
}

server {
    listen 80;
    server_name netbird.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name netbird.example.com;

    ssl_certificate /etc/ssl/certs/netbird.example.com.crt;
    ssl_certificate_key /etc/ssl/private/netbird.example.com.key;

    location / {
        proxy_pass http://netbird_backend;
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
    ServerName netbird.example.com
    Redirect permanent / https://netbird.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName netbird.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/netbird.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/netbird.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:33073/
    ProxyPassReverse / http://127.0.0.1:33073/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend netbird_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/netbird.pem
    redirect scheme https if !{ ssl_fc }
    default_backend netbird_backend

backend netbird_backend
    balance roundrobin
    server netbird1 127.0.0.1:33073 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R netbird:netbird /etc/netbird
sudo chmod 750 /etc/netbird

# Configure firewall
sudo firewall-cmd --permanent --add-port=33073/tcp
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
sudo systemctl status netbird

# View logs
sudo journalctl -u netbird -f

# Monitor resource usage
top -p $(pgrep netbird)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/netbird"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/netbird-backup-$DATE.tar.gz" /etc/netbird /var/lib/netbird

echo "Backup completed: $BACKUP_DIR/netbird-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop netbird

# Restore from backup
tar -xzf /backup/netbird/netbird-backup-*.tar.gz -C /

# Start service
sudo systemctl start netbird
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u netbird -n 100
sudo tail -f /var/log/netbird/netbird.log

# Check configuration
netbird version

# Check permissions
ls -la /etc/netbird
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 33073

# Test connectivity
telnet localhost 33073

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep netbird)

# Check disk I/O
iotop -p $(pgrep netbird)

# Check connections
ss -an | grep 33073
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  netbird:
    image: netbird:latest
    ports:
      - "33073:33073"
    volumes:
      - ./config:/etc/netbird
      - ./data:/var/lib/netbird
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update netbird

# Debian/Ubuntu
sudo apt update && sudo apt upgrade netbird

# Arch Linux
sudo pacman -Syu netbird

# Alpine Linux
apk update && apk upgrade netbird

# openSUSE
sudo zypper update netbird

# FreeBSD
pkg update && pkg upgrade netbird

# Always backup before updates
tar -czf /backup/netbird-pre-update-$(date +%Y%m%d).tar.gz /etc/netbird

# Restart after updates
sudo systemctl restart netbird
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/netbird

# Clean old logs
find /var/log/netbird -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/netbird
```

## Additional Resources

- Official Documentation: https://docs.netbird.org/
- GitHub Repository: https://github.com/netbird/netbird
- Community Forum: https://forum.netbird.org/
- Best Practices Guide: https://docs.netbird.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
