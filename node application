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

