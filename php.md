# Deploy a PHP Website on Ubuntu using Nginx

Follow these step-by-step instructions to set up and deploy a PHP website using Nginx on an Ubuntu VPS.

## Step 1: Install Nginx, PHP, and MySQL
Run the following command to install all required packages:
```sh
sudo apt update && sudo apt upgrade -y
sudo apt install nginx php-fpm php-mysql unzip mysql-server -y
```
### Explanation:
- **nginx** → Web server  
- **php-fpm** → PHP processor for Nginx  
- **php-mysql** → MySQL extension for PHP  
- **mysql-server** → Database server  

## Step 2: Configure PHP
Open the PHP configuration file:
```sh
sudo nano /etc/php/*/fpm/php.ini
```
Find this line and uncomment/change it:
```ini
cgi.fix_pathinfo=0
```
Save and restart PHP-FPM:
```sh
sudo systemctl restart php*-fpm
```

## Step 3: Configure Nginx for PHP
Remove the default Nginx config:
```sh
sudo rm /etc/nginx/sites-enabled/default
```
Create a new Nginx config for your website:
```sh
sudo nano /etc/nginx/sites-available/yourdomain.conf
```
Paste the following configuration:
```nginx
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;  # Change this to your domain or IP

    root /var/www/yourwebsite;  # Set the correct path to your PHP site
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }
}
```
Enable the site:
```sh
sudo ln -s /etc/nginx/sites-available/yourdomain.conf /etc/nginx/sites-enabled/
```
Restart Nginx:
```sh
sudo systemctl restart nginx
```

## Step 4: Upload PHP Website
Move your PHP files to the web directory:
```sh
sudo mkdir -p /var/www/yourwebsite
sudo cp -r /path/to/your/php-files/* /var/www/yourwebsite/
```
Set permissions:
```sh
sudo chown -R www-data:www-data /var/www/yourwebsite
sudo chmod -R 755 /var/www/yourwebsite
```

## Step 5: Configure MySQL (If Needed)
Secure MySQL:
```sh
sudo mysql_secure_installation
```
Create a database:
```sh
sudo mysql -u root -p
```
```sql
CREATE DATABASE mydb;
CREATE USER 'myuser'@'localhost' IDENTIFIED BY 'mypassword';
GRANT ALL PRIVILEGES ON mydb.* TO 'myuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

## Step 6: Test PHP
Create a PHP test file:
```sh
sudo nano /var/www/yourwebsite/info.php
```
Paste the following:
```php
<?php
phpinfo();
?>
```
Access in browser:
```
http://your-server-ip/info.php
```
If you see the PHP information page, PHP is working correctly.

## Step 7: Open Firewall for Nginx
Run:
```sh
sudo ufw allow 'Nginx Full'
sudo ufw enable
sudo ufw status
```

## Step 8: Setup a Domain (Optional)
If you have a domain, update DNS A records to point to your server's public IP.

## Final Check
Open your website in a browser:
```
http://your-server-ip
```
If you configured a domain:
```
http://your-domain.com
```

### Done! Your PHP website is now deployed on Nginx with Ubuntu. 🚀

