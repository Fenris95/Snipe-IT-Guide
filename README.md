# **Complete Guide: Installing Snipe-IT on Ubuntu Server (VMware)**

This guide will walk you through installing **Ubuntu Server on VMware Player/Workstation**, setting up **Snipe-IT**, and configuring all required settings.

---

## **Step 1: Download and Install Ubuntu Server on VMware**

### **1. Download Ubuntu Server ISO**

1. Download **Ubuntu Server ISO** from [Ubuntu’s official website](https://ubuntu.com/download/server).
2. Save the ISO file to your computer.

### **2. Install VMware Workstation or VMware Player**

1. **Download VMware Workstation Player** (free)
2. **Install VMware** following the on-screen instructions.
3. **Launch VMware** after installation.
4. **Create a New Virtual Machine (VM)**:
   - Select "Install from ISO" and choose the Ubuntu Server ISO you downloaded.
   - **Set OS Type**: Linux > Ubuntu 64-bit
   - **Name the Virtual Machine**: Example - `Ubuntu-SnipeIT`
   - **Allocate Storage**: At least **20GB** (use dynamic allocation if needed).
   - **Set Memory**: **4GB+ recommended** (or at least 2GB).
   - **CPU Cores**: At least **2 cores**.
   - **Network Adapter**: Set to **NAT mode** (to share your host’s internet connection).
5. Click **Finish** and **Start the VM**.

---

## **Step 2: Find the VM’s IP Address**

1. **Log into Ubuntu Server in your VMware VM.**
2. Run the following command to find the IP address assigned to your VM:
   ```bash
   ip a
   ```
3. Look for an entry similar to `inet 192.168.x.x/24` under `ens33` (or similar network interface).
4. **Make a note of this IP address**, as you will need it later for configuring **APP\_URL** in Snipe-IT.

---

## **Step 3: Install Prerequisites on Ubuntu Server**

After logging into your Ubuntu Server VM:

1. Update the package lists and upgrade installed packages.
2. Install Apache using the package manager.
3. Enable and start the Apache service.
4. Install PHP and the required extensions.
5. Install and configure MySQL Server.

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y apache2
sudo systemctl enable apache2
sudo systemctl start apache2
sudo apt install php libapache2-mod-php php-mysql php-ldap php-zip php-gd php-mbstring php-curl php-xml php-bcmath php-intl -y
sudo apt install mysql-server -y
```

### **Set Up MySQL Database for Snipe-IT**

1. **Secure MySQL**

   ```bash
   sudo mysql_secure_installation
   ```

   - Set a **strong root password**.
   - Remove anonymous users and test database.
   - Disallow remote root login.

2. **Login to MySQL as root**

   ```bash
   sudo mysql -u root -p
   ```

3. **Create Snipe-IT Database & User**

   ```sql
   CREATE DATABASE snipeit;
   CREATE USER 'snipeituser'@'localhost' IDENTIFIED BY 'your_password';
   GRANT ALL PRIVILEGES ON snipeit.* TO 'snipeituser'@'localhost';
   FLUSH PRIVILEGES;
   EXIT;
   ```

---

## **Step 4: Install Snipe-IT on Ubuntu Server**

### **1. Navigate to Apache Root Directory**

```bash
cd /var/www/html
```

### **2. Clone Snipe-IT from GitHub**

```bash
sudo git clone https://github.com/snipe/snipe-it.git
```

### **3. Move into the Snipe-IT Directory**

```bash
cd /var/www/html/snipe-it 
```

### **4. Copy the Environment File and Edit It**

```bash
sudo cp .env.example .env
sudo nano .env
```

- **Set Database Configuration:**  (update with your details)
  ```
  DB_DATABASE=snipeit
  DB_USERNAME=snipeituser
  DB_PASSWORD=your_password
  ```
- **Set APP\_URL** to your VM's IP (found in Step 2):
  ```bash
  APP_URL=http://192.168.x.x
  ```
- **Save and Exit**: Press `CTRL + X`, then `Y`, then **Enter**.

### **5. Install Composer and Dependencies**

```bash
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
composer install --no-dev --prefer-source
```

> If you see a "permission denied" error, try:

```bash
sudo chown -R $USER:$USER /var/www/html/snipe-it
composer install --no-dev --prefer-source
```

### **6. Generate App Key**

```bash
php artisan key:generate
```

### **7. Set Permissions**

```bash
sudo chown -R www-data:www-data /var/www/html/snipe-it
sudo chmod -R 755 /var/www/html/snipe-it
```

### **8. Set Up Apache Virtual Host**

Create a new Apache configuration file for Snipe-IT:

```bash
sudo nano /etc/apache2/sites-available/snipe-it.conf
```

Add the following configuration:

```
<VirtualHost *:80>
    ServerName your_domain_or_IP
    DocumentRoot /var/www/html/snipe-it/public

    <Directory /var/www/html/snipe-it/public>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/snipe-it_error.log
    CustomLog ${APACHE_LOG_DIR}/snipe-it_access.log combined
</VirtualHost>
```

Enable the new site and the rewrite module:

```bash
sudo a2ensite snipe-it.conf
sudo a2enmod rewrite
sudo systemctl restart apache2
```

---

## **Step 5: Complete the Web-Based Setup**

1. Open a browser on your laptop and visit:
   ```
   http://192.168.202.128
   ```
2. **System Requirements Check:**
   - If all requirements are ✅, click **Next**.
3. **Database Step:** If you already migrated tables earlier, you can **skip this step**.
4. **Admin Setup**:
   - Enter your **admin email, username, and password**.
   - Click **Next**.
5. **Finalize Setup**:
   - Snipe-IT will apply settings and migrations.
   - Log in with your admin account.

**Snipe-IT is now installed and ready to use!**
