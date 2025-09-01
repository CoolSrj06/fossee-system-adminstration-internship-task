# Server Setup – DigitalOcean Droplet (Rocky Linux 10)

## 1. Droplet Creation

- **Provider:** DigitalOcean  
- **Operating System:** Rocky Linux 10 (x64)  
- **Plan Type:** Basic (Shared CPU)  
- **Size Chosen:** 4 GB RAM / 2 vCPUs / 80 GB SSD  
- **Datacenter Region:** <your-region>  
- **Networking:**  
  - IPv4: Assigned by default  
  - IPv6: Not available during creation in this region (can be enabled later if supported)  
- **Authentication:** SSH key-based login (more secure than password).  
- **Hostname:** `rocky-task-droplet`  

---

# Server Hardening – DigitalOcean Droplet

## Access Method
- Since my local machine is Windows, I used **PuTTY** to access the Droplet instead of native `ssh` commands.
- Steps:
  - Installed PuTTY from the official site.
  - Used the Droplet IP as hostname in PuTTY.
  - Logged in initially as `root`.

## Commands Executed

### 1. Create new non-root user
```bash
adduser coolSrj06
passwd coolSrj06
```
### 2.Grant sudo privileges
```bash
usermod -aG wheel coolSrj06
```

### 3. Copy SSH key to new user
```bash
rsync --archive --chown=coolSrj06:coolSrj06 ~/.ssh /home/coolSrj06
```

### 4. Disable root login

Edited the SSH config:
```bash
sudo nano /etc/ssh/sshd_config
```


**Changed:** 
```nginx
PermitRootLogin no
```

**Applied changes:**
```bash
sudo systemctl restart sshd
```

### Reconnect as coolSrj06

![alt text](image.png)

---

# Firewall, System Update, and MariaDB Secure Installation

Here I have described the configuration of the firewall, system package updates, core service installations, and MariaDB hardening on the DigitalOcean Rocky Linux 10 droplet.


## 1. Firewall Installation and Setup

On Rocky Linux 10, `firewalld` was not installed by default, so it had to be added manually.

### Install firewalld
```bash
sudo dnf install firewalld -y
```

### Enable and Start Service
```bash
sudo systemctl enable --now firewalld
```

### Verify Status
```bash
sudo systemctl status firewalld
```
**Expected output:** active (running)

### Configure Firewall Rules
Configured all the firewall rules and recieved a success at each step

---

## 2. Core pakages installed

### 1. Apache HTTP Server

**httpd →** The main web server package.

### 2. PHP (Core + Extensions)

**php** → Base PHP interpreter.

**php-cli →** Command-line PHP interface.

**php-mysqlnd →** MySQL Native Driver for PHP (database connectivity).

**php-gd →** Image processing support.

**php-xml →** XML handling in PHP.

**php-mbstring →** Multibyte string handling (UTF-8, etc.).

**php-json →** JSON support in PHP.

**php-fpm →** FastCGI Process Manager for PHP (better performance with Apache).

### 3. MariaDB (Database Server)

**mariadb-server →** Core database server package.

### 4. Python

**python3 →** Python 3 interpreter.

**python3-pip →** Python package manager.

### 5. Utilities

**unzip →** Extract .zip files.

**wget →** Command-line file downloader.

---

## 3. MariaDB Secure Installation

```bash
sudo mysql_secure_installation
```

### Answers Given
- **Enter current password for root:** Pressed **Enter** (no password set initially).  
- **Switch to unix_socket authentication:** Answered **n** (use password login).  
- **Change the root password:** Answered **Y**, then set a strong root password.  
- **Remove anonymous users:** Answered **Y**.  
- **Disallow root login remotely:** Answered **Y**.  
- **Remove test database:** Answered **Y**.  
- **Reload privilege tables:** Answered **Y**.  


### Result
MariaDB is now secured:
- Root login requires a password.  
- Anonymous accounts removed.  
- Remote root login disabled.  
- Test database removed.  
- Privileges reloaded and active.  





