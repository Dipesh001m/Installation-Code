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







# Resolving .env File Issues in Node.js Deployment on Remote Server

If your .env file is not working when deploying your Node.js application on a remote server, here are some common reasons and solutions:

## 1. Missing dotenv Package
### **Issue:**
Your application may not be loading the .env file because the `dotenv` package is not installed or imported.

### **Solution:**
Ensure that `dotenv` is installed and properly configured.

#### ✅ Steps:
1. Install dotenv in your project:
   ```bash
   npm install dotenv
   ```
2. Import it at the beginning of your `server.js` or `index.js` file:
   ```javascript
   require('dotenv').config();
   ```
3. Restart the server:
   ```bash
   pm2 restart your_app
   ```

---

## 2. Incorrect .env File Path
### **Issue:**
The `.env` file might not be located in the correct directory or not accessible.

### **Solution:**
Ensure the `.env` file is in the root directory of your project (same level as `package.json`).

#### ✅ Steps:
1. Check if the file exists:
   ```bash
   ls -la
   ```
2. If the `.env` file is missing, create one:
   ```bash
   touch .env
   ```

---

## 3. Environment Variables Not Loaded in Production
### **Issue:**
In production, Node.js might not load `.env` automatically.

### **Solution:**
Manually set environment variables using the `export` command in your shell or PM2 ecosystem file.

#### ✅ Steps:
1. Run the following command in the terminal:
   ```bash
   export NODE_ENV=production
   ```
2. Use PM2 to load environment variables:
   Create or edit `ecosystem.config.js`:
   ```javascript
   module.exports = {
     apps: [
       {
         name: "your_app",
         script: "server.js",
         env: {
           NODE_ENV: "production",
           PORT: 3000
         }
       }
     ]
   };
   ```
3. Start the app using:
   ```bash
   pm2 start ecosystem.config.js
   ```

---

## 4. Incorrect Variable Referencing
### **Issue:**
Using incorrect syntax while accessing environment variables in code.

### **Solution:**
Ensure you are using `process.env.VARIABLE_NAME` without spaces.

#### ✅ Example:
```javascript
console.log(process.env.DB_HOST); // Correct
```
❌ Incorrect: `console.log(process.env. DB_HOST);`

---

## 5. File Permissions Issues
### **Issue:**
The `.env` file might not have the correct read permissions.

### **Solution:**
Update permissions to ensure the Node.js process can access it.

#### ✅ Fix:
```bash
chmod 600 .env
```

---

## 6. PM2 Not Loading .env Variables
### **Issue:**
If using PM2, it may not automatically load `.env` variables.

### **Solution:**
Pass the `.env` file explicitly in PM2.

#### ✅ Steps:
1. Start the app with:
   ```bash
   pm2 start server.js --env production
   ```
2. If using `ecosystem.config.js`, define `env_file`:
   ```javascript
   module.exports = {
     apps: [
       {
         name: "your_app",
         script: "server.js",
         env_file: ".env"
       }
     ]
   };
   ```
3. Restart PM2:
   ```bash
   pm2 restart your_app
   ```

---

## 7. Nginx or System Service Not Passing Env Variables
### **Issue:**
If using **Nginx** or a **system service**, it may not pass environment variables.

### **Solution:**
Explicitly define the environment variables in the **systemd service** or **Nginx**.

#### ✅ Example for **systemd Service**:
1. Edit `/etc/systemd/system/yourapp.service`:
   ```ini
   [Service]
   EnvironmentFile=/var/www/LudoGroup_admin/.env
   ExecStart=/usr/bin/node /var/www/LudoGroup_admin/server.js
   ```
2. Restart the service:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl restart yourapp
   ```

---

## 🔎 Final Check
After applying the fixes, restart your application:
```bash
pm2 restart your_app
```
or
```bash
node server.js
```
Then, test if variables are loading:
```javascript
console.log(process.env.YOUR_VARIABLE);
```

---

## ✅ Summary:
✔ Ensure `dotenv` is installed and required in your code  
✔ Check if `.env` exists and has correct permissions  
✔ Use `export VAR=value` if needed in production  
✔ If using PM2, set `env_file: ".env"` in `ecosystem.config.js`  
✔ Restart PM2, Nginx, or system services to apply changes  

🚀 Now your `.env` should work properly!





# Troubleshooting Guide: Git Pull Issues on Remote Server

## **Step 1: Check Git Repository on Remote Server**
SSH into your remote server and navigate to your deployment directory:

```sh
ssh user@remote-server
cd /var/www/LudoGroup_admin
ls -la
```

Check if the `.git` directory exists:

```sh
ls -la .git
```

If `.git` is missing, the repository is not properly initialized. Re-clone the repository:

```sh
git clone git@github.com:your-username/your-repo.git .
```

---

## **Step 2: Verify the Git Remote URL**
Run the following command:

```sh
git remote -v
```

### **Expected Output:**
```
origin  git@github.com:your-username/your-repo.git (fetch)
origin  git@github.com:your-username/your-repo.git (push)
```

If it shows `fatal: not a git repository`, initialize Git and set the remote URL:

```sh
git init
git remote add origin git@github.com:your-username/your-repo.git
git pull origin main
```

---

## **Step 3: Check for SSH Key Issues**
If your remote URL starts with `git@github.com`, Git uses SSH authentication. Ensure your SSH key is set up correctly.

### **Check if the SSH key exists:**

```sh
cat ~/.ssh/id_rsa.pub
```

### **Add the key to GitHub:**
- Go to **GitHub → Settings → SSH Keys**
- Add the **public key (`id_rsa.pub`)**

### **Test SSH connection:**

```sh
ssh -T git@github.com
```

### **Expected Output:**
```
Hi your-username! You've successfully authenticated, but GitHub does not provide shell access.
```

If SSH fails, use HTTPS instead:

```sh
git remote set-url origin https://github.com/your-username/your-repo.git
```

---

## **Step 4: Reset Repository Permissions**
Sometimes, permissions prevent `git pull`. Fix it with:

```sh
sudo chown -R $(whoami):$(whoami) /var/www/LudoGroup_admin
chmod -R 755 /var/www/LudoGroup_admin
```

---

## **Step 5: Switch to the Correct Branch**
Check your current branch:

```sh
git branch
```

If you are on `main`, switch and retry:

```sh
git checkout main
git pull origin main
```

---

## **Step 6: Force Pull if There Are Conflicts**
If `git pull` fails due to local changes:

```sh
git reset --hard HEAD
git pull origin main --force
```

⚠️ **Warning:** This will **discard any local changes!**

---

## **Step 7: Add GitHub Token for Authentication (If Needed)**
If SSH authentication still fails, use a **GitHub Personal Access Token**:

### **Generate a Token:**
- Go to **GitHub → Settings → Developer Settings → Personal Access Tokens**
- Create a token with **repo** permissions

### **Use HTTPS Authentication:**

```sh
git remote set-url origin https://your-token@github.com/your-username/your-repo.git
```

---

## **🎯 Final Check**
Try pulling again:

```sh
git pull origin main
```

If everything works, re-run your **GitHub Actions deployment!** 🚀


