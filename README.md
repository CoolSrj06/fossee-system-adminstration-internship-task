## 📄 `README.md` (progress so far)


# DigitalOcean Keycloak SSO Task

This repository tracks my step-by-step progress for the **DigitalOcean Server Setup with Keycloak SSO** internship screening task.

---

## ✅ Completed So Far

### 1. Droplet Creation
- Created a new Droplet on **DigitalOcean** with:
  - OS: Rocky Linux 10
  - IPv4 + IPv6 enabled
  - Basic 2GB RAM plan
  - Region closest to users
- Configured **SSH key authentication** from Windows.

---

### 2. Server Hardening
- Accessed server using **PuTTY** on Windows.
- Created a new non-root user.
- Granted sudo privileges to new user.
- Disabled direct root login in `/etc/ssh/sshd_config`.
- Verified login works only via the non-root user.

---

At this stage, the following have been completed:

- **Firewall**: Installed, enabled, and configured with rules for HTTP, HTTPS, SSH, and Keycloak (port 8080).
- **System Packages**: Updated to the latest available versions.
- **Core Services**: Apache, PHP (with required extensions), MariaDB, and Python are installed, enabled, and running.
- **Database Security**: MariaDB has been hardened for production use with a root password, anonymous users removed, remote root login disabled, and privileges reloaded.