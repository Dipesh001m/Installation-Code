# **Ludo Application Deployment on a Remote Server Using GitHub Webhooks and Nginx**

## **1️⃣ Overview**
This guide explains how to deploy a Node.js-based Ludo application on a remote server. The application will be automatically updated using **GitHub Webhooks** and served via **Nginx** as a reverse proxy. PM2 will be used for process management.

## **2️⃣ Prerequisites**
- A **remote server** (Ubuntu/Debian/CentOS) with a **public IP**.
- Node.js and NPM installed.
- Git installed (`sudo apt install git` or `sudo yum install git`).
- PM2 installed globally (`npm install -g pm2`).
- Nginx installed (`sudo apt install nginx` or `sudo yum install nginx`).
- A GitHub repository containing the **Ludo application**.

---
## **3️⃣ Step 1: Clone the Ludo Application**
On the remote server, navigate to the deployment directory and clone the application:
```bash
cd /var/www
sudo git clone https://github.com/your-repo/LudoGroup_admin.git
cd LudoGroup_admin
```

Set the correct permissions:
```bash
sudo chown -R $USER:$USER /var/www/LudoGroup_admin
```

---
## **4️⃣ Step 2: Install Dependencies & Configure Environment**
Navigate to the application folder and install dependencies:
```bash
cd /var/www/LudoGroup_admin
npm install
```

Create and configure the `.env` file:
```bash
echo "PORT=3023" > .env
```

---
## **5️⃣ Step 3: Configure GitHub Webhook**
1. Go to your GitHub repository settings.
2. Navigate to **Webhooks** > Click **Add webhook**.
3. Set the **Payload URL** to:
   ```
   http://your_server_ip:3023/github-webhook
   ```
4. Set **Content type** to `application/json`.
5. Select the event trigger: `Just the push event`.
6. Click **Add webhook**.

---
## **6️⃣ Step 4: Modify Ludo Application for Webhook Handling**
Edit `ludoGroup_server.js` to handle webhook requests:
```javascript
const express = require('express');
const { exec } = require('child_process');
require('dotenv').config();

const app = express();
app.use(express.json());

app.post('/github-webhook', (req, res) => {
    console.log('Webhook received from GitHub');
    exec('cd /var/www/LudoGroup_admin && git pull origin master && npm install && pm2 restart ludoGroup_server',
        (err, stdout, stderr) => {
            if (err) {
                console.error(`Deployment Error: ${err.message}`);
                return res.status(500).send('Deployment failed');
            }
            console.log(`stdout: ${stdout}`);
            console.error(`stderr: ${stderr}`);
            res.status(200).send('Deployment successful');
        }
    );
});

const PORT = process.env.PORT || 3023;
app.listen(PORT, () => {
    console.log(`Ludo server running on port ${PORT}`);
});
```

Restart the application:
```bash
pm install
pm start  # Or use PM2 (recommended)
pm2 start ludoGroup_server.js --name ludoGroup_server
pm2 save
```

---
## **7️⃣ Step 5: Configure Nginx as a Reverse Proxy**
### **Install Nginx (if not installed)**
```bash
sudo apt install nginx  # Ubuntu/Debian
sudo yum install nginx  # CentOS/RHEL
```

### **Create an Nginx Configuration File**
```bash
sudo nano /etc/nginx/sites-available/ludo
```

### **Add the Configuration**
```nginx
server {
    listen 80;
    server_name your_server_ip;

    location / {
        proxy_pass http://localhost:3023/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### **Enable the Configuration & Restart Nginx**
```bash
sudo ln -s /etc/nginx/sites-available/ludo /etc/nginx/sites-enabled/
sudo nginx -t  # Test for syntax errors
sudo systemctl restart nginx
```

### **Open Firewall for HTTP Access**
For Ubuntu/Debian (UFW):
```bash
sudo ufw allow 80/tcp
sudo ufw reload
```
For CentOS/RHEL (firewalld):
```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload
```

---
## **8️⃣ Step 6: Test Deployment**
- Open the browser and go to:
  ```
  http://your_server_ip/admin/login
  ```
- Make a small change in the GitHub repository, then push:
  ```bash
  git add .
  git commit -m "Test GitHub webhook deployment"
  git push origin master
  ```
- Check the logs on the remote server:
  ```bash
  pm2 logs ludoGroup_server
  ```

---
## **9️⃣ (Optional) Enable HTTPS with Let's Encrypt**
To secure the application with SSL:
```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d your_domain
```
Now, access your app securely via:
```
https://your_domain/admin/login
```

---
## **🎯 Summary**
| Feature | Without Nginx | With Nginx |
|---------|--------------|------------|
| **Access URL** | `http://your_server_ip:3023/admin/login` | `http://your_server_ip/admin/login` |
| **Removes Port** | ❌ | ✅ |
| **HTTPS (SSL)** | ❌ | ✅ |
| **Security** | ❌ Direct access to Node.js | ✅ Protects backend |
| **Scalability** | ❌ | ✅ Load balancing possible |

✅ **Recommended:** Use **Nginx + PM2** for a production-ready deployment.

---
## **🔹 Final Thoughts**
With this setup:
- The application **auto-updates** on `git push`.
- Nginx **removes the need for ports in the URL**.
- HTTPS can be enabled for **security**.
- PM2 **keeps the app running** even after crashes or reboots.

🚀 **Now your Ludo application is fully deployed and scalable!** Let me know if you need any modifications. 🔥

