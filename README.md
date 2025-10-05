# Configuring Linux Server on EC2

## Overview

This guide documents the complete setup of a production-ready LAMP (Linux, Apache, MySQL, PHP) stack on AWS EC2, culminating in a WordPress installation with SSL/TLS encryption. The server is configured with a custom domain, DNS management using BIND9, and automated SSL certificate renewal through Let's Encrypt.

**Key Components:**
- **Platform:** AWS EC2 (Ubuntu Server 24.04 LTS)
- **Web Server:** Apache2 with Virtual Hosts
- **Database:** MySQL Server
- **CMS:** WordPress
- **Security:** SSL/TLS certificates via Let's Encrypt/Certbot
- **DNS:** BIND9 for local DNS management
- **Domain:** atharvdevops.ddns.net (NoIP DDNS)

**What You'll Learn:**
- Launching and securing an EC2 instance
- Configuring DNS with BIND9
- Setting up Apache2 virtual hosts
- Implementing HTTPS with automated certificate renewal
- Installing and configuring a complete LAMP stack
- Deploying WordPress on your custom domain

---

## Steps Performed:

### Step 1: Launch EC2 Instance
- **AMI Used**: Ubuntu Server 24.04 LTS
- **Instance Type**: t3.micro
- **Key Pair**: Used my own SSH key for authentication, instead of generating a new AWS Key Pair, stored in `~/.ssh/id_rsa`
- **Security Group Rules**:
  - SSH (port 22) open to my IP
  - HTTP (port 80) open to world
  - HTTPS (port 443) open to world
- **Updates & Upgrades**:
  ```bash
  sudo apt update && sudo apt upgrade -y
  sudo passwd # Set password for your user
  ```

---

### Step 2: Domain Name & DNS Configuration
- **Domain**: `atharvdevops.ddns.net` - Got it for free from `https://www.noip.com/`
- **DNS Server**: Configured with `bind9` on the EC2 Instance
  ```bash
  sudo apt install bind9 bind9utils bind9-docs
  ```

#### BIND9 Configuration Files:

1. **Zone Configuration** (`/etc/bind/named.conf.local`)
   ```bash
   zone "atharvdevops.ddns.net" {
       type master;
       file "/etc/bind/db.atharvdevops.ddns.net";
   };
   ```
   Include your zone reference in `/etc/bind/named.conf.local`

2. **Zone File** (`/etc/bind/db.atharvdevops.ddns.net`)
   ```bash
   $TTL    86400
   @       IN      SOA     ns1.example.ddns.net. root.localhost. (
                              177          ; Serial
                            604800         ; Refresh
                             86400         ; Retry
                           2419200         ; Expire
                             86400 )       ; Negative Cache TTL
   ;
   @       IN      NS      ns1.example.ddns.net.
   ns1     IN      A       X.X.X.X
   mail    IN      MX 10   mail.example.ddns.net.
   example.ddns.net.  IN      A       X.X.X.X
   www     IN      A       X.X.X.X
   mail    IN      A       X.X.X.X
   ```
   Place your zone files in `/etc/bind`

**Testing BIND9 Configuration:**
```bash
sudo named-checkconf
sudo named-checkzone atharvdevops.ddns.net /etc/bind/db.atharvdevops.ddns.net
sudo systemctl restart bind9
dig @localhost atharvdevops.ddns.net
```

---

### Step 3: Apache2 Setup & Virtual Host Configuration

1. **Apache2 Installation**
   ```bash
   sudo apt install apache2 -y
   sudo systemctl enable apache2
   sudo systemctl start apache2
   ```

2. **Created Directory for My Site:**
   ```bash
   sudo mkdir -p /var/www/atharvdevops.ddns.net
   sudo chown -R www-data:www-data /var/www/atharvdevops.ddns.net
   sudo chmod 755 /var/www/atharvdevops.ddns.net
   ```

3. **Created a Test File** (`/var/www/atharvdevops.ddns.net/index.html`)
   - Test file is [index.html](apache2-config/index.html)

4. **Create Virtual Host Configuration**
   - Created a new virtual host file:
     ```bash
     sudo nano /etc/apache2/sites-available/atharvdevops.conf
     ```
   - Enable the new virtual host:
     ```bash
     sudo a2ensite atharvdevops.conf
     sudo apache2ctl configtest
     sudo systemctl restart apache2
     ```

5. **Test**
   - Open browser and verified at `http://atharvdevops.ddns.net/`
   
   <img width="1855" height="1006" alt="HTTP Test Screenshot" src="https://github.com/user-attachments/assets/df4d7350-a126-4f0e-9f35-980e39f535e9" />

---

### Step 4: Enable HTTPS (SSL/TLS) with Certbot

1. **Install Certbot**
   ```bash
   sudo apt update
   sudo apt install certbot python3-certbot-apache -y
   ```

2. **Obtain SSL Certificates**
   ```bash
   sudo certbot -d atharvdevops.ddns.net
   ```
   - Enter email for urgent notices (example: atharv5873@gmail.com)
   - Agree to the Terms of Service

   **Result:**
   - Certificate issued and saved at:
     ```bash
     /etc/letsencrypt/live/atharvdevops.ddns.net/fullchain.pem
     /etc/letsencrypt/live/atharvdevops.ddns.net/privkey.pem
     ```
   - Apache virtual host updated automatically to include SSL:
     ```bash
     /etc/apache2/sites-available/atharvdevops.ddns.net-le-ssl.conf
     ```

3. **Test HTTPS**
   - Open browser: `https://atharvdevops.ddns.net/`
    
   <img width="2993" height="1790" alt="HTTPS Test Screenshot" src="https://github.com/user-attachments/assets/b6a930fd-f283-4267-b4ca-c2d3b0b53746" />

4. **Automatic Renewal**
   - Certbot sets up a systemd timer to renew certificates automatically:
     ```bash
     systemctl status certbot.timer
     ```
   - Runs twice daily
   - Renewal simulation test:
     ```bash
     sudo certbot renew --dry-run
     ```
   - Output confirms that renewal works successfully.

---

### Step 5: Completing LAMP Setup

#### 1. PHP Installation
- **Installing PHP:**
  ```bash
  apt install php php-mysql libapache2-mod-php
  ```
- **Testing PHP:**
  - Create a test PHP file: `vim /var/www/atharvdevops.ddns.net/test.php`
    ```bash
    <?php
        phpinfo();
    ?>
    ```
  - Test in browser: `atharvdevops.ddns.net/test.php`

#### 2. MySQL Installation
- **Installing MySQL:**
  ```bash
  apt install mysql-server
  ```
- **Securing MySQL:**
  ```bash
  mysql_secure_installation
  ```
- **Test by Login:**
  ```bash
  mysql -u root
  ```

---

### Step 6: Installing WordPress

1. **Create WordPress Database**
   - Login to MySQL server using `mysql -u root`
     ```bash
     CREATE DATABASE wordpressdb;
     CREATE USER 'wordpressuser'@'localhost' IDENTIFIED BY 'setapasswordhere';
     GRANT ALL PRIVILEGES ON wordpressdb.* TO 'wordpressuser'@'localhost';
     FLUSH PRIVILEGES;
     ```

2. **Download and Install WordPress:**
   ```bash
   cd /tmp
   wget https://wordpress.org/latest.tar.gz
   tar -xzvf latest.tar.gz
   mv wordpress/* /var/www/atharvdevops.ddns.net/
   chown -R www-data:www-data /var/www/atharvdevops.ddns.net
   chmod -R 755 /var/www/atharvdevops.ddns.net
   ```

3. **Configure WordPress**
   - Open in browser: `atharvdevops.ddns.net`
   - Follow the procedure for installing WordPress
   - Login to database using credentials created earlier
   - Set up the WordPress site

4. **Final Result:**
   
   <img width="2999" height="1793" alt="WordPress Installation Complete" src="https://github.com/user-attachments/assets/c65947cf-0438-4b23-a6b6-68ffc0073fd1" />

---

## Summary

This configuration provides a secure, production-ready WordPress site with:
- ✅ Custom domain with DNS management
- ✅ SSL/TLS encryption with automatic renewal
- ✅ Full LAMP stack implementation
- ✅ Proper file permissions and ownership
- ✅ Secure MySQL database configuration

