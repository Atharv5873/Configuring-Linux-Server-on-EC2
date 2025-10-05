# Configuring-Linux-Server-on-EC2

## Steps Performed:

### Step 1: Lanch EC2 Instance
- **AMI Used**: Ubuntu Server 24.04 LTS
- **Instance Type**: t3.micro
- **Key Pair**: Used my own SSH key for authentication, insted of renerating a new AWS Key Pair, stored in `~/.ssh/id_rsa`
- **Security Group Rules**:
  - SSH (port 22) open to my IP
  - HTTP (port 80) open to world
- **Updates & Upgrades**:
  ```bash
  sudo apt update && sudo apt upgrade -y
  sudo passwd #set password for your user
  ```

### Step 2: Domain Name & DNS Configuration:
- **Domain**: `atharvdevops.ddns.net` Got is for free from `https://www.noip.com/`
- **DNS Server**: Configured with `bind9` on the EC2 Instance
  ```bash
  sudo apt install bind9 bind9utils bind9-docs
  ```
#### BIND9 Configuration Files:
1. `name.conf.local`
   ```bash
    zone "atharvdevops.ddns.net" {
	    type master;
	    file "/etc/bind/db.atharvdevops.ddns.net";
    };
   ```
   Include your zone refrence in `/etc/bind/named.conf.local`
2. Zone File
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

Tested BIND9 Configurations:
```bash
sudo named-checkconf
sudo named-checkzone atharvdevops.ddns.net /etc/bind/db.atharvdevops.ddns.net
sudo systemctl restart bind9
dig @localhost atharvdevops.ddns.net
```

### Step 3: Apache2 Setup & Virtual Host Configuration
1. Apache2 Installation
	```bash
 	sudo apt install apache2 -y
 	sudo systemctl enable apache2
	sudo systemctl start apache2
	```
2. Created Directory for my Site:
   ```bash
   sudo mkdir -p /var/www/atharvdevops.ddns.net
   sudo chown -R www-data:www-data /var/www/atharvdevops.ddns.net
   sudo chmod 755 /var/www/atharvdevops.ddns.net
   ```
3. Created a test file `/var/www/atharvdevops.ddns.net/index.html`
	- Test file is [index.html](apache2-config/index.html)

5. Create Virtual Host Configuration
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
6. Test
   Open Browser and Verified at `http://atharvdevops.ddns.net/`
   
   <img width="1855" height="1006" alt="image" src="https://github.com/user-attachments/assets/df4d7350-a126-4f0e-9f35-980e39f535e9" />

### Step 4: Enable HTTPS (SSL/TLS) with Certbot:
1. Install Certbot
   ```bash
   sudo apt update
   sudo apt install certbot python3-certbot-apache -y
   ```
2. Obtain SSL Certificates
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

3. Test HTTPS Open Browser: `https://atharvdevops.ddns.net/`
    
   <img width="2993" height="1790" alt="image" src="https://github.com/user-attachments/assets/b6a930fd-f283-4267-b4ca-c2d3b0b53746" />

4. Automatic Renewal
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

#### Step 5: Compleating LAMP setup:
1. PHP:
	- Installing php:
	  ```bash
	  apt install php php-mysql libapache2-mod-php
	  ```
	- Testing php:
 		- Create a test php file: `vim /var/www/atharvdevops.ddns.net/test.php`
   		```bash
     	<?php
     		phpinfo();
     	?>
     	```
     	- Test in browser : `atharvdevops.ddns.net/test.php`
2. MySql:
   - Installing mysql:
   ```bash
   apt install mysql-server
   ```
   - Securing:
   ```bash
   mysql_secure_installation
   ```
   - Test by login:
   ```bash
   mysql-u root
   ```
