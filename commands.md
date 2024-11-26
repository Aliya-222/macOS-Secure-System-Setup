
# Detailed Steps for User Management, Firewall Configuration, and Automated Backups

This document outlines the entire process of setting up user roles, configuring firewall rules, and automating backups on macOS. Each step is explained with the purpose and expected outcome.

---

## 1. Setting Up User Roles

### Purpose:
To create specific user roles with different levels of access:
- **Admin User**: Full administrative privileges.
- **Developer User**: Elevated privileges for development tasks.
- **Guest User**: Limited access to their own home directory.

### Steps and Commands:
```bash
# Create an admin user
sudo sysadminctl -addUser adminuser -password your_password -admin

# Create a developer user
sudo sysadminctl -addUser devuser -password your_password
sudo dseditgroup -o edit -a devuser -t user admin

# Create a guest user with restricted access
sudo sysadminctl -addUser guestuser -password your_password
sudo chmod 700 /Users/guestuser

# Verify the list of users
dscl . list /Users
```

### Expected Outcome:
- **Admin User**: Full sudo access.
- **Developer User**: Added to the admin group for elevated permissions.
- **Guest User**: Restricted to their home directory.

---

## 2. Configuring Firewall Rules

### Purpose:
To secure the system by allowing only necessary traffic and blocking unauthorized access.

### Steps and Commands:
1. Edit the `pf.conf` file to define custom rules:
   ```bash
   sudo vi /etc/pf.conf
   ```

   Add the following rules:
   ```plaintext
   # Allow SSH only from a specific subnet
   pass in on en0 proto tcp from 10.24.168.2/16 to any port 22

   # Block all other SSH traffic
   block in on en0 proto tcp to any port 22

   # Allow HTTP and HTTPS traffic
   pass in on en0 proto tcp to any port 80
   pass in on en0 proto tcp to any port 443

   # Allow all outgoing traffic
   pass out on en0 all

   # Block all other incoming traffic
   block in on en0 all
   ```

2. Apply and activate the rules:
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
- Check access using `curl`:
   ```bash
   curl http://<your_mac_ip>
   ```

---

## 3. Automating Backups Using Cron

### Purpose:
To ensure data is backed up regularly and automatically.

### Steps and Commands:
1. Install `rsync` for efficient file synchronization:
   ```bash
   brew install rsync
   ```

2. Create a cron job to run nightly backups:
   ```bash
   crontab -e
   ```
   Add the following line to schedule the backup at 2 AM every day:
   ```bash
   0 2 * * * rsync -av --delete /Users/ /backup/
   ```

3. Verify the cron job:
   ```bash
   crontab -l
   ```

### Expected Outcome:
- Backup files will be stored in the `/backup` directory.

---

## 4. Access Restrictions

### Purpose:
To ensure that only authorized users can modify critical files and settings.

### Steps and Commands:
```bash
# Restrict firewall configuration access
sudo chown root:admin /etc/pf.conf
sudo chmod 640 /etc/pf.conf

# Secure cron job directories
sudo chmod 750 /usr/lib/cron
sudo chmod 750 /var/at
```

### Expected Outcome:
- Only admin users can modify firewall and cron settings.

---

## 5. Troubleshooting Notes

### Common Issues:
1. **Blocked Traffic**: Verify using `curl` or `telnet`.
2. **Permission Errors**: Ensure the correct ownership and permissions are set.

---

## 6. Sample Output and Screenshots

### Directory Listing via HTTP:
```html
<!DOCTYPE HTML>
<html lang="en">
<head>
<meta charset="utf-8">
<title>Directory listing for /</title>
</head>
<body>
<h1>Directory listing for /</h1>
<hr>
<ul>
<li><a href="commands.md">commands.md</a></li>
<li><a href="pf.conf">pf.conf</a></li>
</ul>
<hr>
</body>
</html>
```

---

This document serves as a comprehensive guide to replicate the setup. Feel free to adapt and use it for similar configurations.
