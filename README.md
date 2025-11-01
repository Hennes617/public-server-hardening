# Public Server Hardening

## Security

The security implementation follows industry best practices to protect the VPS from common attack vectors. Root login over SSH is disabled to prevent unauthorized administrative access, while SSH key authentication replaces traditional password-based login for stronger security. The default SSH port is changed to a non-standard port to reduce automated attack attempts.

The system maintains current security patches through regular updates and upgrades, complemented by automatic security updates that apply critical patches without manual intervention. A firewall using UFW controls incoming and outgoing traffic, with unnecessary services disabled to minimize the attack surface.

Additional security layers include SFTP for secure file transfers instead of unencrypted FTP, strong password policies enforced through password complexity requirements, and ClamAV antivirus protection for malware detection. User access rights are carefully managed with appropriate privilege levels, and IPv6 is disabled when not needed to prevent additional attack vectors. Fail2ban provides intrusion prevention by blocking repeated failed login attempts.

Malware scanners complement the antivirus software by detecting newer threats including zero-day exploits that traditional antivirus might miss. The scanner updates detection rules frequently to identify emerging threats quickly.

## Password Management

Strong passwords form a critical defense layer. Each password should be long, complex, and unique, incorporating numbers and special characters to resist brute-force attacks. Password managers provide secure storage and generation of these complex passwords, eliminating the need to memorize multiple credentials. Passwords are rotated every three months to maintain security, and no password is reused across different accounts or services.

## File Transfer Security

Secure File Transfer Protocol (SFTP) encrypts all data transmissions including login credentials and file contents. Unlike FTP or FTPS, SFTP authenticates clients before granting system access, preventing man-in-the-middle attacks. The protocol operates over SSH on port 22 by default, providing complete encryption for file operations.

## Monitoring

Grafana provides comprehensive server monitoring with real-time dashboards displaying system metrics including CPU usage, memory consumption, disk space, and network traffic. Prometheus serves as the data collection backend, gathering metrics from the node exporter. Regular security audits using Lynis scan the system weekly for vulnerabilities and configuration issues, with detailed reports stored for review.

## Backups

Daily automated backups preserve critical system configurations and web content. The backup system retains copies for seven days, ensuring recovery options in case of data loss or system compromise. Backup logs track each operation for verification and troubleshooting purposes.

---

This serves as a transparency proof that this public server is managed following security best practices.

## All README.md Files are written by [Hennes617](https://github.com/hennes617). The Ansible playbook.yml with support from [Claude.ai](https://claude.ai)