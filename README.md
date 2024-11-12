---

# Node Security Hardening Guide for BTC and Lightning Node on Ubuntu

This guide provides essential steps to harden the security of a virtual machine running a Bitcoin (BTC) and Lightning Network (LND) node on Ubuntu. These steps help protect your node against unauthorized access, enhance its security posture, and ensure robust logging and monitoring practices.

## Prerequisites

- An existing Ubuntu server with BTC and LND nodes set up and running.
- Root or sudo access to the server.

---

## Hardening Steps

### 1. Update the System

Ensure the system packages are up-to-date:

```bash
sudo apt update && sudo apt upgrade -y
```

### 2. Configure SSH Security

SSH hardening is a critical first step. You’ll set up SSH keys, disable password authentication, change the SSH port, and enforce strong cipher suites.

#### Generate SSH Keys and Copy to Server

If SSH keys are not already set up, you can generate and copy them to the server.

1. Generate an SSH key pair on your local machine:

   ```bash
   ssh-keygen -t ed25519 -C "your_email@example.com"
   ```

2. Copy the public key to your server:

   ```bash
   ssh-copy-id -i ~/.ssh/id_ed25519.pub username@your_server_ip
   ```

   If `ssh-copy-id` is unavailable, manually copy the key:
   
   ```bash
   cat ~/.ssh/id_ed25519.pub
   ```

   Then, on the server, paste the key into `~/.ssh/authorized_keys`.

#### Change the SSH Port and Disable Password Login

1. Open the SSH configuration file:

   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

2. Change the SSH port to a non-standard port (e.g., 2223) and disable password authentication:

   ```ini
   Port 2223
   PasswordAuthentication no
   ```

3. Enforce strong ciphers, key exchange algorithms, and MACs:

   ```ini
   Ciphers aes256-gcm@openssh.com,chacha20-poly1305@openssh.com
   KexAlgorithms curve25519-sha256@libssh.org
   MACs hmac-sha2-512,hmac-sha2-256
   ```

4. Save and close the file, then restart the SSH service:

   ```bash
   sudo systemctl restart ssh
   ```

### 3. Install and Configure Fail2ban

Fail2ban provides protection against brute-force attacks. Configure it to monitor the custom SSH port.

1. Install Fail2ban:

   ```bash
   sudo apt install fail2ban -y
   ```

2. Open the `jail.local` configuration file:

   ```bash
   sudo nano /etc/fail2ban/jail.local
   ```

3. Add the following configuration to protect SSH on port 2223:

   ```ini
   [sshd]
   enabled = true
   port = 2223
   filter = sshd
   logpath = /var/log/auth.log
   maxretry = 5
   ```

4. Save and close the file, then restart Fail2ban:

   ```bash
   sudo systemctl restart fail2ban
   ```

### 4. Configure the Firewall

Use `ufw` to restrict access to only required services and prevent unauthorized access.

1. Enable the firewall:

   ```bash
   sudo ufw enable
   ```

2. Allow the custom SSH port:

   ```bash
   sudo ufw allow 2223/tcp
   ```

3. Allow Bitcoin and Lightning node ports (e.g., 8333 for Bitcoin, 9735 for Lightning):

   ```bash
   sudo ufw allow 8333/tcp
   sudo ufw allow 9735/tcp
   ```

4. If additional services are running, allow their respective ports.

5. Verify the firewall rules:

   ```bash
   sudo ufw status
   ```

### 5. Enable AppArmor for Process Isolation

AppArmor enforces mandatory access control to protect applications and system processes from each other.

1. Ensure AppArmor is installed and enabled:

   ```bash
   sudo apt install apparmor apparmor-utils -y
   sudo systemctl enable apparmor
   sudo systemctl start apparmor
   ```

2. Verify that AppArmor profiles are loaded:

   ```bash
   sudo aa-status
   ```

3. Consider creating custom AppArmor profiles for your BTC and LND node applications to restrict their permissions further.

### 6. Set Up Automatic Security Updates

Enable automatic security updates to keep the system patched and secure.

1. Install the `unattended-upgrades` package:

   ```bash
   sudo apt install unattended-upgrades -y
   ```

2. Enable automatic updates:

   ```bash
   sudo dpkg-reconfigure --priority=low unattended-upgrades
   ```

### 7. Configure Log Monitoring for Security Events

Monitor system logs to detect unusual or suspicious activity on your node.

1. Install `logwatch` for daily email reports on log activity:

   ```bash
   sudo apt install logwatch -y
   ```

2. Configure `logwatch` to send daily summaries of log activity:

   ```bash
   sudo nano /etc/logwatch/conf/logwatch.conf
   ```

   - Set the `MailTo` directive to an email address where you’d like to receive logs.

3. Save the configuration and close the file.

4. To manually test `logwatch`:

   ```bash
   sudo logwatch --output mail --mailto your_email@example.com --detail high
   ```

### 8. Regular Backups

Regularly back up your node data to ensure it can be restored if needed. Use cloud storage, external drives, or an automated backup solution.

---

## Donations

If you appreciate my work, you can make a donation in BTC or via the Lightning Network.

- **BTC donations**: `bc1qy0l39zl7spspzhsuv96c8axnvksypfh8ehvx3e`
- **Lightning Network donations**: `tips@davidebtc.me`

Your support is greatly appreciated!

---

By following this guide, your BTC and Lightning Network node on Ubuntu will be hardened, reducing its attack surface and improving its resilience against common threats and unauthorized access.
