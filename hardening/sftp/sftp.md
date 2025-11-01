# Secure File Transfer Protocol (SFTP) Guide

This guide explains how to implement and use SFTP for secure file transfers to your VPS. SFTP provides encryption for all data transmissions, protecting both authentication credentials and file contents during transfer operations.

## Why SFTP Instead of FTP?

Traditional File Transfer Protocol (FTP) transmits data in plain text, making it vulnerable to interception attacks. Even FTP over TLS (FTPS) only encrypts login credentials while leaving file transmissions partially exposed. Secure File Transfer Protocol (SFTP) addresses these security concerns comprehensively.

SFTP encrypts all data including authentication credentials, file contents, and control commands throughout the entire session. The protocol operates over SSH, inheriting SSH's robust security features including key-based authentication and strong encryption algorithms. This integration prevents man-in-the-middle attacks by requiring client authentication before granting system access.

## Server-Side Configuration

SFTP functionality comes built into SSH, so no additional server software installation is required. The SSH daemon configuration controls SFTP access and permissions.

### Basic SFTP Setup

Verify SSH service is running and properly configured.

```bash
sudo systemctl status ssh
```

Review the SSH configuration file to ensure SFTP subsystem is enabled.

```bash
sudo nano /etc/ssh/sshd_config
```

Locate or add this line to enable the SFTP subsystem:

```
Subsystem sftp /usr/lib/openssh/sftp-server
```

### Restricted SFTP Access (Chroot Jail)

For users who should only access specific directories without shell access, implement SFTP chroot jails. This configuration restricts users to their designated directories while preventing shell command execution.

Create a dedicated group for SFTP-only users.

```bash
sudo groupadd sftpusers
```

Configure the chroot jail in SSH configuration.

```bash
sudo nano /etc/ssh/sshd_config
```

Add these directives at the end of the file:

```
Match Group sftpusers
    ChrootDirectory /var/sftp/%u
    ForceCommand internal-sftp
    AllowTcpForwarding no
    X11Forwarding no
```

Create the directory structure for SFTP users. The chroot directory must be owned by root with specific permissions.

```bash
sudo mkdir -p /var/sftp/username
sudo chown root:root /var/sftp/username
sudo chmod 755 /var/sftp/username
sudo mkdir /var/sftp/username/uploads
sudo chown username:sftpusers /var/sftp/username/uploads
```

Create the SFTP user account with no shell access.

```bash
sudo useradd -g sftpusers -d /uploads -s /usr/sbin/nologin username
sudo passwd username
```

Restart SSH service to apply configuration changes.

```bash
sudo systemctl restart ssh
```

### SFTP Logging

Enable detailed logging for SFTP connections to monitor file transfer activity and detect unauthorized access attempts.

```bash
sudo nano /etc/ssh/sshd_config
```

Set the logging level to capture SFTP operations:

```
LogLevel VERBOSE
```

Monitor SFTP activity through system logs.

```bash
sudo journalctl -u ssh | grep sftp
sudo tail -f /var/log/auth.log | grep sftp
```

## Client-Side Configuration

Multiple SFTP clients provide graphical and command-line interfaces for file transfers. Choose the client that best fits your workflow requirements.

### FileZilla Configuration

FileZilla offers a user-friendly graphical interface for SFTP connections. Download and install FileZilla from the official website.

Launch FileZilla and open the Site Manager. Create a new site with these connection parameters:
- Protocol: Select SFTP - SSH File Transfer Protocol
- Host: Enter your server IP address or domain name
- Port: Use 22 for default SSH port, or your custom SSH port if changed
- Logon Type: Select Key file for SSH key authentication
- User: Enter your SSH username
- Key file: Browse to your private SSH key location

For password authentication (not recommended), select Normal logon type and enter your password. However, key-based authentication provides superior security.

Click Connect to establish the SFTP session. FileZilla displays your local file system on the left and remote server files on the right. Transfer files by dragging and dropping between panels.

### Command-Line SFTP Client

The command-line SFTP client comes pre-installed on Linux and macOS systems. Windows users can access it through Windows Subsystem for Linux or third-party SSH clients.

Connect to the server using SFTP.

```bash
sftp -P 2222 username@server_ip
```

Replace 2222 with your custom SSH port if different. Once connected, use these common commands:

Navigate remote directories:
```
cd /path/to/directory
ls -la
pwd
```

Download files from server to local machine:
```
get remote_file.txt
get -r remote_directory/
```

Upload files from local machine to server:
```
put local_file.txt
put -r local_directory/
```

Create directories on the server:
```
mkdir new_directory
```

Change file permissions:
```
chmod 644 file.txt
```

Exit the SFTP session:
```
bye
```

### WinSCP Configuration (Windows)

WinSCP provides a Windows-native SFTP client with an intuitive interface. Download WinSCP from the official website and install it.

Launch WinSCP and create a new session. Configure these connection settings:
- File protocol: SFTP
- Host name: Your server IP or domain
- Port number: 22 or your custom SSH port
- User name: Your SSH username

For key-based authentication, click Advanced > SSH > Authentication and browse to your private key file (PPK format for PuTTY keys).

Save the session for future use. Click Login to establish the connection. WinSCP displays a dual-pane interface similar to FileZilla, enabling drag-and-drop file transfers.

### Cyberduck Configuration (macOS)

Cyberduck offers a clean macOS-native interface for SFTP connections. Download and install Cyberduck from the official website.

Click Open Connection and select SFTP from the protocol dropdown. Enter these connection details:
- Server: Your server IP or domain
- Port: 22 or your custom SSH port
- Username: Your SSH username
- SSH Private Key: Browse to your private key file

Check "Add to Keychain" to save credentials securely. Click Connect to establish the session. Cyberduck displays remote files in a Finder-style interface with drag-and-drop upload and download capabilities.

## SSH Key Setup for SFTP

SSH key authentication eliminates password transmission, providing stronger security for SFTP connections. Generate key pairs and distribute public keys to servers requiring access.

### Generating SSH Keys

On Linux or macOS, generate an Ed25519 key pair (recommended for strong security with good performance).

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

For systems not supporting Ed25519, use RSA with 4096-bit keys.

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

Save the key pair to the default location or specify a custom path. Set a strong passphrase to protect the private key from unauthorized use if compromised.

On Windows, use PuTTYgen to generate SSH keys. Launch PuTTYgen, select Ed25519 or RSA (4096 bits), and click Generate. Move your mouse to generate randomness. Save both the private key (PPK format) and export the OpenSSH format public key.

### Copying Public Key to Server

Transfer your public key to the server's authorized_keys file.

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub -p 2222 username@server_ip
```

Alternatively, manually copy the public key content to the server:

```bash
cat ~/.ssh/id_ed25519.pub
```

Copy the output, then on the server:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
echo "paste_public_key_here" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

### Testing Key-Based Authentication

Verify SSH key authentication works before disabling password authentication.

```bash
ssh -p 2222 -i ~/.ssh/id_ed25519 username@server_ip
```

Successful login without password prompt confirms key-based authentication is functioning correctly.

## Security Best Practices

Implement these security measures to maximize SFTP protection against unauthorized access and data breaches.

Always use SSH key authentication instead of passwords. Keys provide cryptographic strength that resists brute-force attacks. If passwords must be used, enforce strong complexity requirements and enable automatic account lockouts after failed attempts.

Change the default SSH port from 22 to a non-standard port. While this provides minimal security improvement, it significantly reduces automated attack attempts and log noise from port scanners.

Restrict SFTP access to specific IP addresses when possible. Configure firewall rules or SSH AllowUsers directives to limit connections to known trusted networks.

Disable root login over SSH completely. Create separate administrative accounts with sudo privileges instead. This prevents direct attacks against the root account.

Enable two-factor authentication for SSH connections using tools like Google Authenticator or Duo. This adds an additional security layer beyond key-based authentication.

Implement session timeouts to automatically disconnect idle SFTP connections. This prevents unauthorized access through unattended sessions.

Monitor SFTP logs regularly for suspicious activity including failed authentication attempts, unusual transfer patterns, or access from unexpected locations.

## Troubleshooting Common Issues

SFTP connection problems typically stem from configuration errors, permission issues, or firewall restrictions.

### Connection Refused

If connections are refused, verify SSH service is running and listening on the expected port.

```bash
sudo systemctl status ssh
sudo netstat -tulpn | grep ssh
```

Check firewall rules allow connections on the SSH port.

```bash
sudo ufw status
```

### Permission Denied

Permission errors during file transfers indicate incorrect directory ownership or permissions. Verify the user has write access to the target directory.

```bash
ls -la /path/to/directory
```

Adjust ownership and permissions as needed.

```bash
sudo chown username:groupname /path/to/directory
sudo chmod 755 /path/to/directory
```

### Chroot Jail Issues

Chroot configurations require precise directory ownership and permissions. The chroot directory and all parent directories must be owned by root and not writable by other users.

```bash
sudo chown root:root /var/sftp/username
sudo chmod 755 /var/sftp/username
```

### SSH Key Not Working

If key-based authentication fails, verify the public key is correctly installed in authorized_keys with proper permissions.

```bash
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh
```

Check SSH daemon logs for authentication errors.

```bash
sudo journalctl -u ssh -n 50
```

## Automated File Transfers

Script SFTP operations for automated backups, deployments, or scheduled file synchronization.

### Using SFTP Batch Mode

Create a batch file containing SFTP commands for automated execution.

```bash
echo "cd /remote/directory
put local_file.txt
get remote_file.txt
bye" > sftp_commands.txt
```

Execute the batch file non-interactively.

```bash
sftp -b sftp_commands.txt -P 2222 username@server_ip
```

### Using rsync Over SSH

For efficient synchronization of entire directory trees, use rsync over SSH which provides SFTP-level security with superior performance.

```bash
rsync -avz -e "ssh -p 2222" /local/directory/ username@server_ip:/remote/directory/
```

The -a flag preserves permissions and timestamps, -v provides verbose output, and -z enables compression to reduce transfer time.

---

SFTP provides robust, secure file transfer capabilities essential for protecting sensitive data during transmission. Proper configuration, key-based authentication, and regular monitoring ensure your file transfers remain secure against evolving threats.