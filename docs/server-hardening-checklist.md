# Ubuntu Server Hardening Checklist

This checklist provides a set of common security hardening steps for an Ubuntu server. This is not exhaustive but covers essential practices. Always refer to official documentation and consider your specific security needs.

## 1. Initial Setup & Access Control

* [ ] **Update System:**
    ```bash
    sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y
    ```
* [ ] **Create a Non-Root User with Sudo Privileges:**
    ```bash
    adduser yourusername
    usermod -aG sudo yourusername
    ```
* [ ] **Disable Root Login via SSH:**
    * Edit `/etc/ssh/sshd_config`:
        ```
        PermitRootLogin no
        ```
    * Restart SSH: `sudo systemctl restart sshd`
* [ ] **Configure SSH Key-Based Authentication (Recommended):**
    * Generate SSH keys on your client machine.
    * Copy the public key to `~/.ssh/authorized_keys` on the server for `yourusername`.
    * Optionally disable password authentication in `/etc/ssh/sshd_config`:
        ```
        PasswordAuthentication no
        ChallengeResponseAuthentication no
        UsePAM no # If only using key-based auth and no other PAM modules for SSH
        ```
    * Restart SSH: `sudo systemctl restart sshd`
* [ ] **Change Default SSH Port (Optional, Security through Obscurity):**
    * Edit `/etc/ssh/sshd_config`: `Port YOUR_NEW_SSH_PORT`
    * Update firewall rules for the new port.
    * Restart SSH.
* [ ] **Secure Shared Memory:**
    * Add to `/etc/fstab`: `tmpfs /run/shm tmpfs ro,noexec,nosuid 0 0` (Test thoroughly, might affect some applications).

## 2. Firewall Configuration (UFW)

* [ ] **Install UFW (if not present):** `sudo apt install ufw`
* [ ] **Deny Incoming Traffic by Default:** `sudo ufw default deny incoming`
* [ ] **Allow Outgoing Traffic by Default:** `sudo ufw default allow outgoing`
* [ ] **Allow Necessary Ports:**
    * SSH (your custom port or 22): `sudo ufw allow YOUR_SSH_PORT/tcp`
    * HTTP (80) and HTTPS (443) - *Only if not using Cloudflare Tunnel exclusively. If Cloudflared connects to a local Docker port, that port does NOT need to be opened in UFW.*
        ```bash
        # sudo ufw allow http
        # sudo ufw allow https
        ```
    * Other necessary services.
* [ ] **Enable UFW:** `sudo ufw enable`
* [ ] **Check Status:** `sudo ufw status verbose`

## 3. Intrusion Detection & Prevention

* [ ] **Install Fail2Ban:**
    ```bash
    sudo apt install fail2ban
    ```
* [ ] **Configure Fail2Ban:**
    * Copy default jail config: `sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`
    * Edit `jail.local` to enable protection for SSH and other services:
        ```ini
        [sshd]
        enabled = true
        port = YOUR_SSH_PORT
        # bantime = 1h
        # findtime = 10m
        # maxretry = 3
        ```
    * Restart Fail2Ban: `sudo systemctl restart fail2ban`
    * Check status: `sudo fail2ban-client status sshd`

## 4. System Services & Software

* [ ] **Remove Unnecessary Services/Software:**
    * List installed packages: `apt list --installed`
    * Remove unneeded packages: `sudo apt remove package-name`
* [ ] **Disable Unused Services:**
    * List services: `systemctl list-unit-files --type=service`
    * Disable: `sudo systemctl disable service-name`
* [ ] **Regularly Check for Listening Ports:**
    ```bash
    sudo ss -tulnp
    sudo netstat -tulnp
    ```

## 5. Logging and Monitoring

* [ ] **Ensure System Logging is Active (`rsyslog` or `journald`):** Usually enabled by default.
* [ ] **Configure Log Rotation (`logrotate`):** Usually pre-configured. Check `/etc/logrotate.conf` and `/etc/logrotate.d/`.
* [ ] **(Optional) Centralized Logging:** Consider sending logs to a remote syslog server or SIEM.
* [ ] **(Optional) System Auditing (`auditd`):**
    ```bash
    sudo apt install auditd
    # Configure rules in /etc/audit/rules.d/
    sudo systemctl enable auditd
    sudo systemctl start auditd
    ```

## 6. Kernel Hardening (sysctl)

* [ ] **Create a custom sysctl config file (e.g., `/etc/sysctl.d/99-security.conf`):**
    ```ini
    # IP Spoofing protection
    net.ipv4.conf.all.rp_filter = 1
    net.ipv4.conf.default.rp_filter = 1

    # Ignore ICMP broadcast requests
    net.ipv4.icmp_echo_ignore_broadcasts = 1

    # Disable source packet routing
    net.ipv4.conf.all.accept_source_route = 0
    net.ipv6.conf.all.accept_source_route = 0
    net.ipv4.conf.default.accept_source_route = 0
    net.ipv6.conf.default.accept_source_route = 0

    # Ignore send redirects
    net.ipv4.conf.all.send_redirects = 0
    net.ipv4.conf.default.send_redirects = 0

    # Disable ICMP redirects
    net.ipv4.conf.all.accept_redirects = 0
    net.ipv6.conf.all.accept_redirects = 0
    net.ipv4.conf.default.accept_redirects = 0
    net.ipv6.conf.default.accept_redirects = 0
    net.ipv4.conf.all.secure_redirects = 0
    net.ipv4.conf.default.secure_redirects = 0

    # Enable TCP SYN Cookie Protection
    net.ipv4.tcp_syncookies = 1
    ```
* [ ] **Apply settings:** `sudo sysctl -p /etc/sysctl.d/99-security.conf` (or `sudo sysctl -p` to load all files)

## 7. Filesystem & Permissions

* [ ] **Check for World-Writable Files:** `find / -xdev -type d -perm -0002 -print` (investigate results).
* [ ] **Check for SUID/SGID Binaries:** `find / -xdev \( -perm -4000 -o -perm -2000 \) -print` (investigate unusual entries).
* [ ] **Set Strong Default Permissions (umask):** Consider setting a stricter umask (e.g., `027`) in `/etc/profile` or `/etc/login.defs`.

## 8. Regular Maintenance

* [ ] **Schedule Automatic Updates (unattended-upgrades):**
    ```bash
    sudo apt install unattended-upgrades
    sudo dpkg-reconfigure --priority=low unattended-upgrades
    ```
* [ ] **Regularly Review Logs.**
* [ ] **Perform Security Audits Periodically.**
* [ ] **Backup Critical Data (including server configuration).**

This checklist provides a starting point. Always research and understand each setting before applying it.