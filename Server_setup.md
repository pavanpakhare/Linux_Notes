# Setting Up a Linux Server: Web, File, and Database Services

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Initial Server Setup](#initial-server-setup)
3. [Web Server Setup](#web-server-setup)
4. [File Server Setup](#file-server-setup)
5. [Database Server Setup](#database-server-setup)
6. [Security Hardening](#security-hardening)
7. [Maintenance and Monitoring](#maintenance-and-monitoring)

## Prerequisites

- A Linux server (physical or cloud instance)
- SSH access with root/sudo privileges
- Basic Linux command line knowledge
- Domain name (optional for web server)

## Initial Server Setup

### 1. Connect to Your Server
```bash
ssh root@your_server_ip
# Or for non-root users:
ssh username@your_server_ip
```

### 2. Update the System
```bash
sudo apt update && sudo apt upgrade -y  # Debian/Ubuntu
sudo yum update -y                     # CentOS/RHEL
sudo dnf update -y                     # Fedora
```

### 3. Create a New User (Recommended)
```bash
sudo adduser serveradmin
sudo usermod -aG sudo serveradmin  # Add to sudo group
```

### 4. Set Up SSH Key Authentication
On your local machine:
```bash
ssh-keygen -t rsa -b 4096
ssh-copy-id serveradmin@your_server_ip
```

On server, disable password authentication:
```bash
sudo nano /etc/ssh/sshd_config
# Change:
PasswordAuthentication no
PermitRootLogin no
```
Then restart SSH:
```bash
sudo systemctl restart sshd
```

### 5. Set Up a Basic Firewall
```bash
sudo ufw allow OpenSSH  # Debian/Ubuntu
sudo ufw enable

# Or for CentOS/RHEL:
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --reload
```

## Web Server Setup

### Option 1: Apache
```bash
sudo apt install apache2 -y  # Debian/Ubuntu
sudo yum install httpd -y    # CentOS/RHEL

# Start and enable
sudo systemctl start apache2
sudo systemctl enable apache2

# Verify
curl localhost
```

### Option 2: Nginx
```bash
sudo apt install nginx -y    # Debian/Ubuntu
sudo yum install nginx -y   # CentOS/RHEL

# Start and enable
sudo systemctl start nginx
sudo systemctl enable nginx

# Verify
curl localhost
```

### Configure Virtual Host (Apache)
```bash
sudo mkdir -p /var/www/example.com/html
sudo chown -R $USER:$USER /var/www/example.com/html
sudo chmod -R 755 /var/www/example.com

# Create sample page
echo "<h1>Welcome to Example.com!</h1>" | sudo tee /var/www/example.com/html/index.html

# Create config file
sudo nano /etc/apache2/sites-available/example.com.conf

# Add this content:
<VirtualHost *:80>
    ServerAdmin admin@example.com
    ServerName example.com
    ServerAlias www.example.com
    DocumentRoot /var/www/example.com/html
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Enable the site:
```bash
sudo a2ensite example.com.conf
sudo a2dissite 000-default.conf
sudo systemctl restart apache2
```

### Configure Server Block (Nginx)
```bash
sudo nano /etc/nginx/sites-available/example.com

# Add this content:
server {
    listen 80;
    listen [::]:80;

    root /var/www/example.com/html;
    index index.html;

    server_name example.com www.example.com;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Enable the site:
```bash
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
sudo nginx -t  # Test configuration
sudo systemctl restart nginx
```

### Install PHP (Optional)
```bash
sudo apt install php libapache2-mod-php php-mysql -y
sudo systemctl restart apache2

# Test with:
echo "<?php phpinfo(); ?>" | sudo tee /var/www/example.com/html/info.php
```

## File Server Setup

### Option 1: Samba (Windows/Linux/Mac)
```bash
sudo apt install samba -y
sudo mkdir -p /srv/samba/share
sudo chown nobody:nogroup /srv/samba/share
sudo chmod 777 /srv/samba/share

# Configure Samba
sudo nano /etc/samba/smb.conf

# Add at bottom:
[share]
    comment = Shared Folder
    path = /srv/samba/share
    browsable = yes
    guest ok = yes
    read only = no
    create mask = 0777

# Restart Samba
sudo systemctl restart smbd
```

### Option 2: NFS (Linux-to-Linux)
```bash
sudo apt install nfs-kernel-server -y
sudo mkdir -p /srv/nfs/share
sudo chown nobody:nogroup /srv/nfs/share
sudo chmod 777 /srv/nfs/share

# Configure exports
sudo nano /etc/exports
# Add:
/srv/nfs/share *(rw,sync,no_subtree_check)

# Apply changes
sudo exportfs -a
sudo systemctl restart nfs-kernel-server
```

### Option 3: SFTP (Secure File Transfer)
```bash
sudo groupadd sftpusers
sudo usermod -aG sftpusers serveradmin

# Configure SSH for SFTP
sudo nano /etc/ssh/sshd_config

# Add at bottom:
Match Group sftpusers
    ChrootDirectory /home
    ForceCommand internal-sftp
    AllowTcpForwarding no
    X11Forwarding no

# Create directory
sudo mkdir -p /home/serveradmin/files
sudo chown serveradmin:sftpusers /home/serveradmin/files
sudo chmod 700 /home/serveradmin/files

# Restart SSH
sudo systemctl restart sshd
```

## Database Server Setup

### Option 1: MySQL
```bash
sudo apt install mysql-server -y

# Secure installation
sudo mysql_secure_installation

# Create database and user
sudo mysql
CREATE DATABASE example_db;
CREATE USER 'example_user'@'localhost' IDENTIFIED BY 'strong_password';
GRANT ALL PRIVILEGES ON example_db.* TO 'example_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### Option 2: PostgreSQL
```bash
sudo apt install postgresql postgresql-contrib -y

# Create database and user
sudo -u postgres psql
CREATE DATABASE example_db;
CREATE USER example_user WITH PASSWORD 'strong_password';
GRANT ALL PRIVILEGES ON DATABASE example_db TO example_user;
\q
```

### Option 3: MongoDB
```bash
sudo apt install mongodb -y

# Start and enable
sudo systemctl start mongodb
sudo systemctl enable mongodb

# Create admin user
mongo
use admin
db.createUser({
  user: "admin",
  pwd: "strong_password",
  roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
})
exit
```

## Security Hardening

### 1. Keep System Updated
```bash
sudo apt update && sudo apt upgrade -y
```

### 2. Configure Automatic Updates
```bash
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure unattended-upgrades
```

### 3. Install Fail2Ban
```bash
sudo apt install fail2ban -y
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

### 4. Set Up SSH Security
```bash
sudo nano /etc/ssh/sshd_config
# Change:
Port 2222  # Change from default 22
PermitRootLogin no
MaxAuthTries 3
LoginGraceTime 20
```

### 5. Install and Configure UFW
```bash
sudo apt install ufw -y
sudo ufw allow 2222  # Your SSH port
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
```

## Maintenance and Monitoring

### Basic Monitoring Commands
```bash
top                  # Process monitoring
htop                 # Enhanced process viewer
df -h                # Disk space
free -h              # Memory usage
netstat -tuln        # Open ports
journalctl -xe       # System logs
```

### Set Up Log Rotation
```bash
sudo nano /etc/logrotate.conf
# Adjust settings as needed
```

### Backup Strategy
```bash
# Example backup script
sudo nano /usr/local/bin/backup.sh

#!/bin/bash
# Backup web files
tar -czf /backups/web-$(date +%Y%m%d).tar.gz /var/www
# Backup databases
mysqldump -u root -p'password' --all-databases > /backups/db-$(date +%Y%m%d).sql
# Keep only last 7 backups
find /backups/* -mtime +7 -exec rm {} \;

# Make executable
sudo chmod +x /usr/local/bin/backup.sh

# Add to cron
sudo crontab -e
0 3 * * * /usr/local/bin/backup.sh
```

### Monitoring with Cockpit (Web UI)
```bash
sudo apt install cockpit -y
sudo systemctl enable --now cockpit.socket
# Access at https://your_server_ip:9090
```

## Final Steps

1. Test all services:
   - Access web server via browser
   - Connect to file shares from clients
   - Test database connections

2. Document your setup:
   - Keep records of usernames/passwords
   - Note configuration changes
   - Document backup procedures

3. Set up regular maintenance:
   - Schedule updates
   - Monitor logs
   - Test backups regularly

4. Consider setting up:
   - SSL certificates (Let's Encrypt)
   - Monitoring tools (Nagios, Zabbix)
   - Centralized logging (ELK Stack)

Remember to always:
- Use strong passwords
- Keep software updated
- Monitor system logs
- Regularly test your backups
- Follow the principle of least privilege for user access
