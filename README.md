
# User Management, Firewall Configuration, and Automated Backup Setup

This repository documents the steps, commands, and troubleshooting processes I used to configure user permissions, firewall rules, and automated backups on macOS. It includes practical examples, screenshots, and attached configuration files.

---

## Table of Contents

1. [Setting Up User Roles](#1-setting-up-user-roles)
2. [Configuring Firewall Rules](#2-configuring-firewall-rules)
3. [Automating Backups Using Cron](#3-automating-backups-using-cron)
4. [Access Restrictions](#4-access-restrictions)
5. [Troubleshooting and Results](#5-troubleshooting-and-results)

---

## 1. Setting Up User Roles

### Commands Used:
```bash
sudo sysadminctl -addUser adminuser -password your_password -admin
sudo sysadminctl -addUser devuser -password your_password
sudo dseditgroup -o edit -a devuser -t user admin
sudo sysadminctl -addUser guestuser -password your_password
sudo chmod 700 /Users/guestuser
dscl . list /Users
```

### Purpose:
- **Admin User**: Full permissions.
- **Developer User**: Admin privileges for development tasks.
- **Guest User**: Limited to home directory access.

---

## 2. Configuring Firewall Rules

### Configuration Steps:
1. Open the `pf.conf` file:
   ```bash
   sudo vi /etc/pf.conf
   ```
2. Add the following rules:
   ```plaintext
   # Allow SSH from specific subnet
   pass in on en0 proto tcp from 10.24.168.2/16 to any port 22

   # Block all other SSH traffic
   block in on en0 proto tcp to any port 22

   # Allow HTTP/HTTPS traffic
   pass in on en0 proto tcp to any port 80
   pass in on en0 proto tcp to any port 443

   # Allow outgoing traffic
   pass out on en0 all

   # Block all other incoming traffic
   block in on en0 all
   ```
3. Apply and verify the rules:
   ```bash
   sudo pfctl -f /etc/pf.conf
   sudo pfctl -e
   sudo pfctl -sr
   ```

### Testing:
- Start a local server:
   ```bash
   python3 -m http.server 80
   ```
- Test connectivity:
   ```bash
   curl http://<your_mac_ip>
   curl https://<your_mac_ip>
   ```

### Screenshots:
- **HTTP Test:**
  ![HTTP Test](./HTTP_port.png)
- **HTTPS Test:**
  ![HTTPS Test](./HTTPS_port.png)

---

## 3. Automating Backups Using Cron

### Steps:
1. Install `rsync`:
   ```bash
   brew install rsync
   ```
2. Add a cron job:
   ```bash
   crontab -e
   ```
   Example cron job for nightly backups at 2 AM:
   ```
   0 2 * * * rsync -av --delete /Users/ /backup/
   ```
3. Verify:
   ```bash
   crontab -l
   ```

---

## 4. Access Restrictions

### Commands:
```bash
# Restrict firewall configurations
sudo chown root:admin /etc/pf.conf
sudo chmod 640 /etc/pf.conf

# Secure cron job directories
sudo chmod 750 /usr/lib/cron
sudo chmod 750 /var/at
```

---

## 5. Troubleshooting and Results

### Issues Encountered:
1. **Blocked Ports**: Verified with `curl` and `telnet`.
2. **Permissions Issues**: Resolved by ensuring proper ownership and restrictions.

### Results:
- Users and roles work as expected.
- Firewall rules applied successfully.
- Backups automated and verified.

### Attachments:
- [commands.md](./commands.md)
- [pf.conf](./pf.conf)
- Screenshots:
  - HTTP and HTTPS tests included.

---

## Next Steps:
This project demonstrates robust user and system configurations. Feel free to clone, modify, or extend for your own use. Pull requests are welcome!

---

### Repository Usage:
1. Clone the repo:
   ```bash
   git clone <repository-url>
   ```
2. Follow the documentation to replicate the setup.

