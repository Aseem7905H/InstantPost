 
---

# README: WordPress Automation Deployment with Apache, MySQL, and WP-CLI

## Overview
This project automates the deployment of WordPress websites with Apache, MySQL, and WP-CLI. The solution utilizes a Virtual Machine (VM) running on Oracle VirtualBox with Ubuntu Server. The automation script sets up Apache virtual hosts, installs WordPress, configures MySQL, and automates the entire process using WP-CLI. Additionally, a simple web page is created to input domains and run the automation script.

The solution is scalable and designed to deploy multiple WordPress sites on a single VM, making it easier for manufacturers or startups to deploy dedicated e-commerce platforms.

## Technologies Used
- **Ubuntu Server (18.04 or later)**
- **Apache2**
- **MySQL**
- **PHP**
- **WordPress**
- **WP-CLI**
- **Bash scripting**
- **PHP for the web interface**
- **Oracle VirtualBox for VM management**

## VM Configuration
- **Operating System**: Ubuntu Server 20.04 LTS
- **RAM**: 2GB (or more)
- **Disk**: 8GB Virtual Disk
- **Software**:
  - Apache2
  - MySQL
  - PHP
  - Python
  - Node.js
  - WordPress
  - WP-CLI (WordPress Command-Line Interface)

## Step-by-Step Implementation

### Step 1: Set up Apache Virtual Host on Ubuntu Server
1. **Install Apache:**
   ```bash
   sudo apt update
   sudo apt install apache2 -y
   ```

2. **Create Virtual Host Configuration:**
   - Navigate to the Apache configuration directory:
     ```bash
     cd /etc/apache2/sites-available/
     ```
   - Create a new configuration file for `local.example.com`:
     ```bash
     sudo nano local.example.com.conf
     ```
   - Add the following content:
     ```apache
     <VirtualHost *:80>
         ServerAdmin webmaster@localhost
         ServerName local.example.com
         DocumentRoot /var/www/local.example.com
         ErrorLog ${APACHE_LOG_DIR}/error.log
         CustomLog ${APACHE_LOG_DIR}/access.log combined
     </VirtualHost>
     ```

3. **Enable the Virtual Host and Restart Apache:**
   ```bash
   sudo a2ensite local.example.com.conf
   sudo systemctl restart apache2
   ```

### Step 2: Set up WordPress on Virtual Machine
1. **Install PHP, MySQL, and Other Dependencies:**
   ```bash
   sudo apt install php libapache2-mod-php php-mysql mysql-server php-curl php-xml php-mbstring -y
   ```

2. **Set up MySQL Database for WordPress:**
   ```bash
   sudo mysql -u root -p
   CREATE DATABASE wordpress;
   CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'password';
   GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';
   FLUSH PRIVILEGES;
   exit;
   ```

3. **Download and Install WordPress:**
   ```bash
   cd /var/www/
   sudo wget https://wordpress.org/latest.tar.gz
   sudo tar -xvzf latest.tar.gz
   sudo mv wordpress local.example.com
   sudo chown -R www-data:www-data local.example.com/
   ```

4. **Configure WordPress (wp-config.php):**
   ```bash
   cd /var/www/local.example.com
   sudo cp wp-config-sample.php wp-config.php
   sudo nano wp-config.php
   # Edit the following values:
   define('DB_NAME', 'wordpress');
   define('DB_USER', 'wpuser');
   define('DB_PASSWORD', 'password');
   ```

5. **Test Installation:**
   Open a browser and navigate to `http://local.example.com` to complete the WordPress installation via the web interface.

### Step 3: Configure /etc/hosts for Local Domain Resolution
1. **Edit the /etc/hosts File:**
   ```bash
   sudo nano /etc/hosts
   # Add the following line:
   127.0.0.1       local.example.com
   ```

### Step 4: Automate the Setup using WP-CLI
1. **Install WP-CLI:**
   ```bash
   curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
   sudo chmod +x wp-cli.phar
   sudo mv wp-cli.phar /usr/local/bin/wp
   ```

2. **Create a Bash Script (setup_wordpress.sh):**
   Create the `setup_wordpress.sh` script to automate WordPress installation for a given domain:
   ```bash
   sudo nano /var/www/setup_wordpress.sh
   ```
   Add the following content:
   ```bash
   #!/bin/bash
   DOMAIN=$1
   DB_NAME=$2
   DB_USER=$3
   DB_PASSWORD=$4

   # Set up Virtual Host
   echo "
   <VirtualHost *:80>
       ServerAdmin webmaster@localhost
       ServerName $DOMAIN
       DocumentRoot /var/www/$DOMAIN
       ErrorLog ${APACHE_LOG_DIR}/error.log
       CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>
   " > /etc/apache2/sites-available/$DOMAIN.conf

   # Enable site and restart Apache
   a2ensite $DOMAIN.conf
   systemctl restart apache2

   # Download and set up WordPress
   cd /var/www/
   wget https://wordpress.org/latest.tar.gz
   tar -xvzf latest.tar.gz
   mv wordpress $DOMAIN
   chown -R www-data:www-data $DOMAIN/

   # Configure wp-config.php
   cp /var/www/$DOMAIN/wp-config-sample.php /var/www/$DOMAIN/wp-config.php
   sed -i "s/database_name_here/$DB_NAME/" /var/www/$DOMAIN/wp-config.php
   sed -i "s/username_here/$DB_USER/" /var/www/$DOMAIN/wp-config.php
   sed -i "s/password_here/$DB_PASSWORD/" /var/www/$DOMAIN/wp-config.php

   # Install WordPress via WP-CLI
   cd /var/www/$DOMAIN
   wp core install --url=$DOMAIN --title="New Site" --admin_user="admin" --admin_password="adminpassword" --admin_email="admin@example.com"
   ```

3. **Make the Script Executable:**
   ```bash
   chmod +x /var/www/setup_wordpress.sh
   ```

4. **Run the Automation Script:**
   ```bash
   ./setup_wordpress.sh local.test.com wordpress wpuser password
   ```

### Step 5: Create a Simple Web Interface for Domain Input
1. **Create an HTML Form:**
   Create a simple form that accepts a domain name from the user.
   ```bash
   sudo nano /var/www/html/index.html
   ```
   Example form:
   ```html
   <!DOCTYPE html>
   <html>
   <head>
       <title>Domain Setup</title>
   </head>
   <body>
       <form action="setup_wordpress.php" method="POST">
           Domain: <input type="text" name="domain"><br>
           <input type="submit" value="Set Up">
       </form>
   </body>
   </html>
   ```

2. **Create PHP Script for Handling Form Submission:**
   ```bash
   sudo nano /var/www/html/setup_wordpress.php
   ```
   Example PHP code:
   ```php
   <?php
   if ($_SERVER["REQUEST_METHOD"] == "POST") {
       $domain = $_POST['domain'];
       $command = "bash /var/www/setup_wordpress.sh $domain wordpress wpuser password";
       shell_exec($command);
       echo "Setup Complete for $domain";
   }
   ?>
   ```

### Step 6: Test the Web Interface
- Open a browser and go to `http://<VM-IP>/index.html`. Enter a domain name and submit the form to trigger the WordPress setup automation.

 
