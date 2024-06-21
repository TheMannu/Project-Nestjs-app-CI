### Step 1: Set Up the EC2 Instance
1. **Launch an EC2 Instance:**
   - Go to the AWS Management Console.
   - Launch a new EC2 instance with Ubuntu.
   - Choose an appropriate instance type (e.g., t2.micro for free tier).
   - Configure the security group to allow SSH (port 22) and HTTP (port 80) access.

2. **Connect to the EC2 Instance:**
   - Use SSH to connect to your instance.
     
sh
     ssh -i "your-key-pair.pem" ubuntu@your-ec2-public-dns


3. **Update the System:**
   
sh
   sudo apt update
   sudo apt upgrade -y


### Step 2: Install Node.js and NestJS
1. **Install Node.js:**
   
sh
   curl -sL https://deb.nodesource.com/setup_16.x | sudo -E bash -
   sudo apt install -y nodejs


2. **Install NestJS CLI:**
   
sh
   sudo npm install -g @nestjs/cli


### Step 3: Create and Deploy the NestJS Application
1. **Create a New NestJS Application:**
   
sh
   nest new hello-world
   cd hello-world


2. **Start the NestJS Application:**
   
sh
   npm run start


3. **Install PM2 to Manage the Application:**
   
sh
   sudo npm install -g pm2
   pm2 start dist/main.js --name "hello-world"
   pm2 startup
   pm2 save


### Step 4: Set Up NGINX as a Reverse Proxy
1. **Install NGINX:**
   
sh
   sudo apt install nginx


2. **Configure NGINX:**
   
sh
   sudo nano /etc/nginx/sites-available/default

   Replace the contents with:
   
nginx
   server {
       listen 80;
       server_name your_domain_or_IP;

       location / {
           proxy_pass http://localhost:3000;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
           proxy_set_header Host $host;
           proxy_cache_bypass $http_upgrade;
       }
   }


3. **Restart NGINX:**
   
sh
   sudo systemctl restart nginx


### Step 5: Set Up CI/CD with GitHub Actions
1. **Create a GitHub Repository:**
   - Create a new repository on GitHub and push your NestJS application to it.

2. **Create a GitHub Actions Workflow:**
   - In your GitHub repository, create a .github/workflows/deploy.yml file with the following content:
   
yaml
   name: Deploy to EC2

   on:
     push:
       branches:
         - main

   jobs:
     build:
       runs-on: ubuntu-latest

       steps:
       - name: Checkout code
         uses: actions/checkout@v2

       - name: Set up Node.js
         uses: actions/setup-node@v2
         with:
           node-version: '16'

       - name: Install dependencies
         run: npm install

       - name: Build the project
         run: npm run build

       - name: Deploy to EC2
         uses: appleboy/ssh-action@master
         with:
           host: ${{ secrets.EC2_HOST }}
           username: ubuntu
           key: ${{ secrets.EC2_KEY }}
           script: |
             cd hello-world
             git pull origin main
             npm install
             npm run build
             pm2 restart hello-world


3. **Add Secrets to GitHub Repository:**
   - Go to your GitHub repository settings.
   - Add EC2_HOST (your EC2 public DNS), EC2_USER (usually ubuntu), and EC2_KEY (your private SSH key content) in the Secrets section.
