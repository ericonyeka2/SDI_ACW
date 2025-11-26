# 601249 – Server Configuration Summary

This document explains the configuration of the virtual machine, the purpose behind each major configuration. The setup aims to provide a secure multi-user environment, controlled file transfer, a correctly permissioned static website, and a reverse-proxied Docker application.

---

## 1. User Accounts and Access Separation

Three operational accounts were created to support different roles:

- **marketing**
- **design**
- **audit**

Each account was assigned permissions based on the principle of least privilege: 

- **marketing** requires the ability to upload website content but should not access other users’ files.  
  - Added to the `webedit` group for write access to `/srv/www`.
  - Restricted to SFTP only by placing the account in the `sftponly` group.

- **design** manages project files unrelated to the website.  
  - Given full access to its own home directory, which contains nested project folders.
  - Denied access to the web server directory to prevent accidental modification.

- **audit** needs read-only visibility across project and website files.  
  - Restricted to SFTP only.
  - Granted **read but not write** access to marketing and design directories using ACLs (`getfacl` / `setfacl`).

This structure ensures clear separation of duties and protects data integrity between teams.

---

## 2. File Transfer Configuration

SFTP was configured to give users secure, encrypted access with controlled capabilities:

- The `Match Group sftponly` rule in `sshd_config` forces SFTP-only access and blocks shell usage for marketing and audit.
- SSH keys were deployed into each account’s `~/.ssh/authorized_keys` to provide key-based authentication and prevent password-based access.
- The design directory structure was created to match operational requirements:

/home/design/project_rocket/{cad,render}
/home/design/project_cheese/{research,tests}

These directories provide isolated spaces for each project while ensuring other accounts cannot modify them.

---

## 3. Web Server (Nginx) Configuration

A static web server was deployed using Nginx with the root located at:

/srv/www/student

This directory contains a text file that returns the 6-digit student number.  
Permissions were assigned so that:

- The **webedit** group can update content.
- Marketing and audit can access the site according to their assigned privileges.

Serving static files through Nginx ensures predictable behaviour for automated tests and prevents accidental write access from other system users.

---

## 4. Docker Application Reverse Proxy

A Docker container running a Node.js application was built and configured to start automatically.

To expose the application securely, Nginx was used as a reverse proxy:

- Requests to the main hostname serve the static site.  
- Requests using the `docker.` subdomain are forwarded to the container at `http://172.18.0.10:3000`.

This approach isolates the web service from the application runtime and ensures both services run on port 80 while remaining logically separated.

---

## 5. Maintenance Commands

 **Nginx**
* sudo systemctl status nginx
* sudo systemctl reload nginx
* sudo nginx -t

**Docker**
* sudo docker ps
* sudo docker logs sdi_web
* sudo docker restart sdi_web

**User and Permission Checks**
* ls -l /home
* getfacl -R /home/design
* getfacl -R /srv/www
* tail -n 20 /var/log/auth.log
