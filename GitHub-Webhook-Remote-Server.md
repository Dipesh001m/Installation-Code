# Deploying Node.js Application on a Remote Server using GitHub Actions CI/CD Pipeline

## **1. Prerequisites**
Before setting up GitHub Actions, ensure that the following prerequisites are met:

- A remote server with SSH access.
- Node.js and PM2 installed on the remote server.
- A GitHub repository containing your Node.js application.
- SSH key-based authentication configured between GitHub Actions and the remote server.

## **2. Configure Your Remote Server**
### **Install Node.js and PM2**
Run the following commands on your remote server:
```bash
sudo apt update && sudo apt install -y nodejs npm
npm install -g pm2
```

### **Set Up SSH Access for GitHub Actions**
1. Generate an SSH key on your local machine:
   ```bash
   ssh-keygen -t rsa -b 4096 -C "github-actions"
   ```
2. Copy the public key (`~/.ssh/id_rsa.pub`) to the remote server:
   ```bash
   ssh-copy-id -i ~/.ssh/id_rsa user@remote_server_ip
   ```
3. Add the **private key (`id_rsa`)** as a **GitHub Secret** (e.g., `SSH_PRIVATE_KEY`).

## **3. Create GitHub Actions Workflow**
In your GitHub repository, create a workflow file: `.github/workflows/deploy.yml`

```yaml
name: Deploy Node.js App

on:
  push:
    branches:
      - main  # Change branch if needed

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 18  # Adjust as required

      - name: Install Dependencies
        run: npm install

      - name: Run Tests (Optional)
        run: npm test  # Skip if not applicable

      - name: Deploy to Remote Server
        uses: appleboy/ssh-action@v0.1.6
        with:
          host: ${{ secrets.SSH_HOST }}
          username: ${{ secrets.SSH_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /var/www/LudoGroup_admin
            git pull origin main
            npm install
            pm2 restart ludoGroup_server.js || pm2 start ludoGroup_server.js
```

## **4. Add GitHub Secrets**
Go to **GitHub Repository → Settings → Secrets and Variables → Actions → New Repository Secret**, then add:

| Secret Name        | Value (Example)                   |
|--------------------|---------------------------------|
| `SSH_HOST`        | `your_server_ip`                |
| `SSH_USER`        | `your_ssh_user`                 |
| `SSH_PRIVATE_KEY` | `Contents of id_rsa`            |

## **5. Push Code & Trigger Deployment**
After setting up the workflow, push your code to the `main` branch to trigger deployment:
```bash
git add .
git commit -m "Deploy Node.js app via GitHub Actions"
git push origin main
```

## **6. Verify Deployment**
To check if your application is running correctly on the remote server:
```bash
pm2 list
pm2 logs ludoGroup_server.js
```

## **Conclusion**
With this setup, your Node.js application will be automatically deployed to the remote server whenever you push changes to the `main` branch. This ensures a seamless and automated deployment process using GitHub Actions CI/CD pipeline.




# Deploying a Node.js Application on a Remote Server Using Nginx and GitHub Actions CI/CD

## 1. Setup Remote Server
Ensure your remote server (e.g., AWS EC2, VPS, DigitalOcean, etc.) has:
- **Node.js** and **NPM** installed
- **PM2** installed to manage the application
- **Nginx** installed for reverse proxy
- **SSH access** using a private key

## 2. Configure Nginx Reverse Proxy
Edit the Nginx configuration file (`/etc/nginx/sites-available/default` or `/etc/nginx/nginx.conf`):

```nginx
server {
    listen 80;
    server_name your_domain_or_server_ip;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Restart Nginx:
```bash
sudo systemctl restart nginx
```

## 3. Setup SSH Access for GitHub Actions
### Generate SSH Key
Run the following command on your local machine:
```bash
ssh-keygen -t rsa -b 4096 -C "github-actions-deploy"
```

### Configure Server
Copy the **public key** (`~/.ssh/id_rsa.pub`) and add it to `/home/your_user/.ssh/authorized_keys` on the remote server.

### Configure GitHub Secrets
Copy the **private key** (`~/.ssh/id_rsa`) and add it as a **GitHub Secret** (e.g., `SSH_PRIVATE_KEY`).

## 4. Create GitHub Actions Workflow
Create a `.github/workflows/deploy.yml` file in your repository:

```yaml
name: Deploy Node.js App

on:
  push:
    branches:
      - main  # Change to your deployment branch

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Install Dependencies
        run: npm install

      - name: Build Application
        run: npm run build

      - name: Deploy to Remote Server
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.SERVER_IP }}
          username: ${{ secrets.SSH_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /var/www/LudoGroup_admin
            git pull origin main
            npm install
            pm2 restart ludoGroup_server.js
            sudo systemctl restart nginx
```

## 5. Add GitHub Secrets
Go to **GitHub → Your Repo → Settings → Secrets and Variables → Actions → New Repository Secret**

Add the following secrets:
- `SERVER_IP` → Your server's IP
- `SSH_USER` → Your SSH username
- `SSH_PRIVATE_KEY` → Your private key

## 6. Push Code to Trigger Deployment
Push changes to the **`main` branch**:
```bash
git add .
git commit -m "Deploying Node.js app"
git push origin main
```
GitHub Actions will **automatically deploy** your app on the remote server.

## 7. Verify Deployment
Check if the application is running:
```bash
pm2 status
curl http://your_server_ip
```

## Conclusion
Your Node.js application is now deployed on a remote server using **GitHub Actions, Nginx, and PM2** for process management. 🚀

