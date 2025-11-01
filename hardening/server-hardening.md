# Server Hardening Guide

This comprehensive guide covers essential security measures to protect your VPS from common threats and vulnerabilities.

## 1. System Updates and Package Management

Begin by updating the package cache and upgrading all installed packages to their latest versions. This ensures all security patches are applied immediately.

```bash
sudo apt update && sudo apt upgrade -y
```

Configure automatic security updates to maintain protection without manual intervention. Install the unattended-upgrades package to enable this functionality.

```bash
sudo apt install unattended-upgrades apt-listchanges
sudo dpkg-reconfigure -plow unattended-upgrades
```

## 2. SSH Security Configuration

SSH represents a primary attack vector, so proper configuration is critical. Start by backing up the original configuration file before making changes.

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
```

### SSH Key Authentication

Generate SSH keys on your local machine using strong encryption. Copy the public key to the server before disabling password authentication.

```bash
# On local machine
ssh-keygen -t ed25519 -C "your_email@example.com"
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@server_ip
```

### SSH Configuration

Edit the SSH configuration file to implement security hardening measures.

```bash
sudo nano /etc/ssh/sshd_config
```

Set the following parameters:
- Port: Change from default 22 to a custom port (e.g., 2222) to reduce automated attacks
- PermitRootLogin: Set to no to disable direct root access
- PasswordAuthentication: Set to no to enforce key-based authentication only
- PubkeyAuthentication: Set to yes to enable SSH key login
- MaxAuthTries: Set to 3 to limit brute force attempts
- X11Forwarding: Set to no unless specifically needed
- Protocol: Set to 2 to use only the secure SSH protocol version
- ClientAliveInterval: Set to 300 to disconnect idle sessions
- ClientAliveCountMax: Set to 2 for inactive connection handling

Restart SSH service after configuration changes.

```bash
sudo systemctl restart ssh
```

## 3. Firewall Configuration

UFW provides a straightforward interface for managing firewall rules. Configure default policies to deny incoming traffic while allowing outgoing connections.

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Allow necessary services explicitly before enabling the firewall.

```bash
sudo ufw allow 2222/tcp comment 'SSH'
sudo ufw allow 80/tcp comment 'HTTP'
sudo ufw allow 443/tcp comment 'HTTPS'
sudo ufw enable
```

## 4. Fail2ban Installation

Fail2ban monitors log files for repeated failed authentication attempts and automatically blocks offending IP addresses.

```bash
sudo apt install fail2ban
```

Create a local configuration file to protect SSH on your custom port.

```bash
sudo nano /etc/fail2ban/jail.local
```

Add the following configuration:

```
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 3

[sshd]
enabled = true
port = 2222
logpath = /var/log/auth.log
backend = systemd
```

Restart fail2ban to apply the configuration.

```bash
sudo systemctl restart fail2ban
```

## 5. Secure File Transfer Protocol (SFTP)

SFTP encrypts all data including credentials and file contents, unlike traditional FTP. Configure your FTP client to use SFTP instead.

For FileZilla or similar clients, use these connection settings:
- Protocol: SFTP
- Host: Your server IP
- Port: 22 (or your custom SSH port)
- Logon Type: Key file
- User: Your username
- Key file: Path to your private SSH key

## 6. Password Policy and Management

Implement strong password requirements for all accounts. Passwords should meet these criteria:
- Minimum 16 characters length
- Combination of uppercase and lowercase letters
- Include numbers and special characters
- No dictionary words or personal information
- Unique for each account
- Changed every three months

Consider using password managers like Bitwarden, 1Password, or KeePassXC to generate and store complex passwords securely.

## 7. Antivirus Installation (ClamAV)

ClamAV provides open-source antivirus protection for Linux systems. Note that ClamAV requires at least 2GB of free RAM to operate effectively.

```bash
sudo apt install clamav clamav-daemon
```

Update virus definitions immediately after installation.

```bash
sudo freshclam
```

Configure automatic definition updates.

```bash
sudo nano /etc/clamav/freshclam.conf
```

Run manual scans on demand or schedule them via cron jobs.

```bash
sudo clamscan -r --bell -i /
```

## 8. Malware Scanner Integration

Malware scanners complement antivirus software by detecting newer threats including zero-day exploits. Install additional scanning tools for comprehensive protection.

```bash
sudo apt install rkhunter chkrootkit
```

Configure and run initial scans.

```bash
sudo rkhunter --update
sudo rkhunter --check
```

## 9. User Rights Management

Proper user privilege management prevents unauthorized access to sensitive system resources. Create separate accounts for different administrative tasks rather than sharing the root account.

### Creating User Groups

```bash
sudo groupadd developers
sudo groupadd administrators
```

### Creating Users with Appropriate Privileges

```bash
sudo useradd -m -G developers -s /bin/bash username
sudo passwd username
```

### Granting Sudo Access

For users requiring administrative capabilities, add them to the sudo group rather than granting full root access.

```bash
sudo usermod -aG sudo username
```

### Setting Directory Permissions

Apply appropriate read, write, and execute permissions to directories based on user roles.

```bash
sudo chmod 750 /var/www/sensitive_directory
sudo chown -R username:www-data /var/www/sensitive_directory
```

Regularly audit user accounts to identify and remove unnecessary or compromised accounts.

```bash
cut -d: -f1 /etc/passwd
```

## 10. IPv6 Configuration

Disable IPv6 if not actively used to prevent attackers from exploiting it as an alternative attack vector.

```bash
sudo nano /etc/sysctl.conf
```

Add these lines to the configuration file:

```
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```

Apply the changes immediately.

```bash
sudo sysctl -p
```

Verify IPv6 is disabled by checking the value.

```bash
cat /proc/sys/net/ipv6/conf/all/disable_ipv6
```

A return value of 1 confirms IPv6 is disabled.

## 11. Service Management

Disable unnecessary services to reduce the attack surface. List all running services and disable those not required for your operations.

```bash
systemctl list-unit-files --type=service --state=enabled
sudo systemctl disable service-name
sudo systemctl stop service-name
```

## 12. Security Auditing

Regular security audits identify vulnerabilities and misconfigurations. Lynis provides comprehensive system scanning capabilities.

```bash
sudo apt install lynis
sudo lynis audit system
```

Review the audit report located in /var/log/lynis.log and address identified issues. Schedule weekly automated scans via cron jobs.

```bash
sudo crontab -e
```

Add this line for weekly Sunday night scans:

```
0 3 * * 0 lynis audit system --quiet > /var/log/lynis-audit-$(date +\%Y\%m\%d).log 2>&1
```

## 13. Log Monitoring

Monitor system logs regularly for suspicious activity. Configure log rotation to manage disk space while maintaining audit trails.

```bash
sudo apt install logrotate
```

Review authentication logs for failed login attempts or unusual access patterns.

```bash
sudo journalctl -u ssh -n 100
sudo tail -f /var/log/auth.log
```

## 14. Application Updates

Keep all installed applications current with the latest security patches. For web applications like WordPress, enable automatic updates when possible.

For Debian/Ubuntu systems:

```bash
sudo apt update
sudo apt list --upgradable
sudo apt upgrade
```

For CentOS/RHEL systems:

```bash
sudo yum check-update
sudo yum update
```

Configure cron jobs to automate update checks and notifications, though manual review before applying updates is recommended for critical systems.

---

Following these hardening measures significantly improves your VPS security posture. Regular maintenance, monitoring, and timely updates ensure ongoing protection against evolving threats.