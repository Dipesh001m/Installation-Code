# Deploying a Full-Stack Node.js Application on a Remote VPS

## 1. Install Required Dependencies

### 1.1 Install Node.js and npm
```sh
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
node -v
npm -v
```

### 1.2 Install Git
```sh
sudo apt install -y git
git --version
```

### 1.3 Install PM2 (Process Manager for Node.js)
```sh
sudo npm install -g pm2
pm2 startup
```

### 1.4 Install MongoDB (If Using a Local Database)
```sh
sudo apt install -y mongodb-org
sudo systemctl enable mongod
sudo systemctl start mongod
```
Check if MongoDB is running:
```sh
sudo systemctl status mongod
```

### 1.5 Install Nginx (For Reverse Proxy)
```sh
sudo apt install -y nginx
sudo systemctl enable nginx
sudo systemctl start nginx
```

Configure reverse proxy in `/etc/nginx/sites-available/default`:
```nginx
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```
Restart Nginx:
```sh
sudo systemctl restart nginx
```

## 2. Set Up Application and Install Dependencies

### 2.1 Clone the GitHub Repository
```sh
ssh user@your-vps-ip
cd /var/www
git clone git@github.com:your-username/your-repo.git LudoGroup_admin
cd LudoGroup_admin
```

### 2.2 Install Node.js Dependencies
```sh
npm install
```

### 2.3 Configure Environment Variables
Create a `.env` file:
```sh
nano .env
```
Example content:
```
PORT=3000
MONGO_URI=mongodb://localhost:27017/your_database
JWT_SECRET=your-secret-key
```

### 2.4 Build the Frontend (If Using React, Angular, or Vue)
```sh
cd client
npm install
npm run build
cp -r build/* ../public/
```

## 3. Set File Permissions

Ensure the user has proper ownership and permissions:
```sh
sudo chown -R $(whoami):$(whoami) /var/www/LudoGroup_admin
sudo chmod -R 755 /var/www/LudoGroup_admin
```

## 4. Start and Manage the Application

### 4.1 Start the Application Using PM2
```sh
pm2 start ludoGroup_server.js --name "ludo-app"
pm2 save
```
Check if it’s running:
```sh
pm2 list
```
Restart after updates:
```sh
pm2 restart ludo-app
```

## 5. Automate Deployment Using GitHub Actions

### 5.1 Add SSH Key for GitHub Actions
Generate SSH key:
```sh
ssh-keygen -t rsa -b 4096 -C "your-email@example.com"
```
Copy public key to VPS:
```sh
cat ~/.ssh/github-actions-key.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

### 5.2 Configure GitHub Actions Workflow
Create `.github/workflows/deploy.yml`:
```yaml
name: Deploy Node.js App

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup SSH and Deploy
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.SSH_HOST }}
          username: ${{ secrets.SSH_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /var/www/LudoGroup_admin
            git pull origin main
            npm install
            pm2 restart ludo-app
```

### 5.3 Add Secrets in GitHub
Go to **GitHub Repo** → **Settings** → **Secrets and variables** → **Actions**:

| Secret Name       | Value |
|-------------------|-------|
| `SSH_HOST`       | `your-vps-ip` |
| `SSH_USER`       | `your-vps-username` |
| `SSH_PRIVATE_KEY` | **(Paste your SSH private key)** |

## 6. Final Checks

✅ **Test the app manually:**
```sh
curl -I http://your-vps-ip:3000
```
✅ **Check logs for errors:**
```sh
pm2 logs
```
✅ **Ensure GitHub Actions deploys automatically when pushing to `main`.**

---

### 🎯 Now Your Full-Stack Node.js App is Successfully Deployed on VPS! 🚀





# Debugging MySQL Connection Issues in Node.js Application

## **Environment Variables Setup**
Ensure your application is configured with the following environment variables:

```ini
DB_HOST=localhost
DB_USER=root
DB_PASS=orion
DB_NAME=admin
```

If your application throws an "admin.admin database not found" error, follow these steps to debug and fix the issue:

---

## **✅ Step 1: Verify MySQL Service is Running**
Run the following command to check if MySQL is active:

```sh
sudo systemctl status mysql
```

If it’s not running, start it with:

```sh
sudo systemctl start mysql
```

To enable MySQL on startup:

```sh
sudo systemctl enable mysql
```

---

## **✅ Step 2: Confirm the Database Exists**
Login to MySQL:

```sh
mysql -u root -p
```
(Enter password: `orion`)

Then, list the databases:

```sql
SHOW DATABASES;
```

If `admin` is missing, create it:

```sql
CREATE DATABASE admin;
```

Check the tables:

```sql
USE admin;
SHOW TABLES;
```

If no tables exist, check your migration or seed scripts.

---

## **✅ Step 3: Check Your MySQL Connection in Node.js**
Ensure your application is correctly connecting to MySQL. Update or add the following in your Node.js project:

### **Using `mysql2` Library (Recommended)**
Install `mysql2` if not installed:

```sh
npm install mysql2
```

Then, in your application:

```js
const mysql = require('mysql2');

const connection = mysql.createConnection({
    host: process.env.DB_HOST || 'localhost',
    user: process.env.DB_USER || 'root',
    password: process.env.DB_PASS || 'orion',
    database: process.env.DB_NAME || 'admin'
});

connection.connect((err) => {
    if (err) {
        console.error('❌ Database connection failed:', err.message);
        return;
    }
    console.log('✅ Connected to MySQL database.');
});

module.exports = connection;
```

Run your app:

```sh
node server.js
```

If it fails, check for errors and update them accordingly.

---

## **✅ Step 4: Verify `.env` is Loaded**
If your Node.js app isn't recognizing `.env`, install `dotenv`:

```sh
npm install dotenv
```

Then, update your main app file (e.g., `server.js`):

```js
require('dotenv').config();
```

Ensure `.env` is in the root of your project.

To test, print the variables:

```js
console.log("Database Name:", process.env.DB_NAME);
```

If it logs `undefined`, your `.env` file might not be loading.

---

## **✅ Step 5: Check for Port or User Restrictions**
If the database is hosted remotely or in a **Docker container**, ensure:

### **Your MySQL allows remote connections**  
- Check `my.cnf` (`/etc/mysql/my.cnf` or `/etc/mysql/mysql.conf.d/mysqld.cnf`):

```sh
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

- Ensure `bind-address = 0.0.0.0`
- Restart MySQL:

```sh
sudo systemctl restart mysql
```

### **Your MySQL user has proper permissions**
Run:

```sql
GRANT ALL PRIVILEGES ON admin.* TO 'root'@'%' IDENTIFIED BY 'orion';
FLUSH PRIVILEGES;
```

---

## **🚀 Final Check**
1. Restart your application:
   ```sh
   node server.js
   ```
2. If errors persist, check logs:
   ```sh
   tail -f logs/error.log
   ```
3. If needed, manually connect:
   ```sh
   mysql -h localhost -u root -p
   ```

---

### **🎯 Summary**
✅ Ensure MySQL is running  
✅ Verify the `admin` database exists  
✅ Check Node.js MySQL connection code  
✅ Confirm `.env` is loading properly  
✅ Allow external connections if necessary  

Let me know if the issue persists! 🚀


