# Production Deployment Guide

This guide provides step-by-step instructions for deploying the MEAN stack application to AWS EC2 using Docker and GitHub Actions CI/CD.

## Prerequisites Checklist

- [ ] GitHub account created
- [ ] Docker Hub account created
- [ ] AWS account with EC2 access
- [ ] SSH key pair generated (`.pem` file)
- [ ] Domain name (optional)

## Part 1: GitHub Repository Setup

### 1.1 Create GitHub Repository

1. Go to https://github.com/new
2. Repository name: `crud-dd-task-mean-app` (or your choice)
3. Set to Public or Private
4. Do NOT initialize with README (code already has one)
5. Click "Create repository"

### 1.2 Push Code to GitHub

```bash
cd c:\Users\karth\Downloads\crud-dd-task-mean-app\crud-dd-task-mean-app

# Initialize git (if not already initialized)
git init

# Add all files
git add .

# Commit
git commit -m "Initial commit: MEAN stack app with Docker and CI/CD"

# Add remote (replace with your repo URL)
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO_NAME.git

# Push to main branch
git branch -M main
git push -u origin main
```

## Part 2: Docker Hub Setup

### 2.1 Create Docker Hub Account

1. Go to https://hub.docker.com/signup
2. Sign up with email
3. Verify your email

### 2.2 Create Repositories

1. Click "Create Repository"
2. Create repository: `mean-app-backend`
   - Visibility: Public
   - Click "Create"
3. Create repository: `mean-app-frontend`
   - Visibility: Public
   - Click "Create"

### 2.3 Generate Access Token

1. Click on your username → Account Settings
2. Go to "Security" tab
3. Click "New Access Token"
4. Description: "GitHub Actions CI/CD"
5. Access permissions: Read, Write, Delete
6. Click "Generate"
7. **COPY THE TOKEN** (you won't see it again!)

## Part 3: AWS EC2 Setup

### 3.1 Launch EC2 Instance

1. Sign in to AWS Console
2. Navigate to EC2 Dashboard
3. Click "Launch Instance"

**Instance Configuration:**
- **Name:** MEAN-App-Server
- **AMI:** Ubuntu Server 22.04 LTS (Free tier eligible)
- **Instance type:** t2.medium (recommended) or t2.small (minimum)
- **Key pair:** Create new or select existing
  - If creating new: Download `.pem` file and store securely
- **Network settings:**
  - Allow SSH traffic from: Your IP
  - Allow HTTP traffic from: Internet (0.0.0.0/0)
  - Allow HTTPS traffic from: Internet (optional)
- **Storage:** 20 GB gp3 (minimum)

4. Click "Launch instance"
5. Wait for instance state: "Running"
6. Note the **Public IPv4 address**

### 3.2 Connect to EC2

**On Windows (using PowerShell or Git Bash):**
```bash
# Move to where your .pem file is located
cd path\to\your\key

# Set proper permissions (Git Bash)
chmod 400 your-key.pem

# Connect via SSH
ssh -i your-key.pem ubuntu@YOUR_EC2_PUBLIC_IP
```

**On macOS/Linux:**
```bash
chmod 400 /path/to/your-key.pem
ssh -i /path/to/your-key.pem ubuntu@YOUR_EC2_PUBLIC_IP
```

### 3.3 Install Docker on EC2

Once connected, run:

```bash
# Update package index
sudo apt-get update

# Install prerequisites
sudo apt-get install -y ca-certificates curl gnupg lsb-release

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add ubuntu user to docker group
sudo usermod -aG docker ubuntu

# Install Docker Compose V2
sudo apt-get install -y docker-compose-plugin

# Verify installation
docker --version
docker compose version
```

**IMPORTANT:** Log out and log back in for group changes to take effect:
```bash
exit
# Then SSH back in
ssh -i your-key.pem ubuntu@YOUR_EC2_PUBLIC_IP
```

### 3.4 Create Project Directory

```bash
# Create project directory
mkdir -p ~/crud-dd-task-mean-app
cd ~/crud-dd-task-mean-app
```

### 3.5 Create docker-compose.yml for Production

```bash
nano docker-compose.yml
```

Paste the following (**REPLACE `YOUR_DOCKERHUB_USERNAME`**):

```yaml
version: '3.8'

services:
  mongodb:
    image: mongo:6
    container_name: mean-mongodb
    restart: unless-stopped
    environment:
      - MONGO_INITDB_DATABASE=dd_db
    volumes:
      - mongodb_data:/data/db
    networks:
      - mean-network

  backend:
    image: YOUR_DOCKERHUB_USERNAME/mean-app-backend:latest
    container_name: mean-backend
    restart: unless-stopped
    environment:
      - MONGODB_URI=mongodb://mongodb:27017/dd_db
      - PORT=8080
    depends_on:
      - mongodb
    networks:
      - mean-network

  frontend:
    image: YOUR_DOCKERHUB_USERNAME/mean-app-frontend:latest
    container_name: mean-frontend
    restart: unless-stopped
    depends_on:
      - backend
    networks:
      - mean-network

  nginx:
    image: nginx:alpine
    container_name: mean-nginx-proxy
    restart: unless-stopped
    ports:
      - "80:80"
    volumes:
      - ./nginx-proxy.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - frontend
      - backend
    networks:
      - mean-network

networks:
  mean-network:
    driver: bridge

volumes:
  mongodb_data:
```

Save and exit (Ctrl+O, Enter, Ctrl+X)

### 3.6 Create Nginx Proxy Configuration

```bash
nano nginx-proxy.conf
```

Paste:

```nginx
server {
    listen 80;
    server_name localhost;

    location /api/ {
        proxy_pass http://backend:8080/api/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }

    location / {
        proxy_pass http://frontend:80/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Save and exit (Ctrl+O, Enter, Ctrl+X)

## Part 4: Configure GitHub Secrets

### 4.1 Add Docker Hub Credentials

1. Go to your GitHub repository
2. Click Settings → Secrets and variables → Actions
3. Click "New repository secret"

**Add these secrets:**

| Name | Value |
|------|-------|
| `DOCKER_USERNAME` | Your Docker Hub username |
| `DOCKER_PASSWORD` | The access token you generated earlier |

### 4.2 Add EC2 Credentials

**Add these secrets:**

| Name | Value | Example |
|------|-------|---------|
| `EC2_HOST` | Your EC2 public IP | `54.123.45.67` |
| `EC2_USERNAME` | SSH username | `ubuntu` |
| `EC2_SSH_KEY` | Content of your `.pem` file | Full content including `-----BEGIN RSA PRIVATE KEY-----` and `-----END RSA PRIVATE KEY-----` |

**To get EC2_SSH_KEY value:**

**On Windows:**
```powershell
Get-Content your-key.pem | clip
# Now paste in GitHub secret
```

**On macOS/Linux:**
```bash
cat your-key.pem
# Copy the entire output including BEGIN and END lines
```

## Part 5: Update GitHub Actions Workflow

The workflow file `.github/workflows/deploy.yml` should already be in your repo. If you need to update your Docker Hub username:

1. Open `.github/workflows/deploy.yml`
2. The workflow uses `${{ secrets.DOCKER_USERNAME }}` (no changes needed if secrets are set correctly)

## Part 6: First Deployment

### 6.1 Trigger CI/CD Pipeline

The pipeline will trigger automatically on any push to `main` branch.

**Option 1: Make a small change**
```bash
# On your local machine
cd c:\Users\karth\Downloads\crud-dd-task-mean-app\crud-dd-task-mean-app

# Make a small change (e.g., update README)
echo "" >> README.md

git add .
git commit -m "Trigger first deployment"
git push origin main
```

**Option 2: Manual workflow trigger**
1. Go to GitHub → Actions tab
2. Select "CI/CD Pipeline" workflow
3. Click "Run workflow"

### 6.2 Monitor Deployment

1. Go to GitHub → Actions tab
2. Click on the latest workflow run
3. Watch the progress of "Build and Push Docker Images" and "Deploy to EC2" jobs
4. Wait for both jobs to complete (green checkmark)

### 6.3 Verify Deployment on EC2

SSH back into your EC2 instance:

```bash
ssh -i your-key.pem ubuntu@YOUR_EC2_PUBLIC_IP

cd ~/crud-dd-task-mean-app

# Check if containers are running
docker ps

# You should see 4 containers:
# - mean-mongodb
# - mean-backend
# - mean-frontend
# - mean-nginx-proxy

# Check logs if needed
docker-compose logs -f backend
docker-compose logs -f frontend
```

### 6.4 Access the Application

Open your browser and navigate to:
```
http://YOUR_EC2_PUBLIC_IP
```

You should see the Angular application!

Test the API directly:
```
http://YOUR_EC2_PUBLIC_IP/api/tutorials
```

Should return: `[]` (empty array)

## Part 7: Testing the Application

1. **Create a Tutorial:**
   - Click "Add" button
   - Fill in title and description
   - Click "Submit"

2. **View Tutorials:**
   - Should appear in the list

3. **Edit Tutorial:**
   - Click on a tutorial
   - Modify details
   - Click "Update"

4. **Search:**
   - Use the search box
   - Click "Search"

5. **Delete:**
   - Click "Delete" on a tutorial
   - Or "Remove All" to delete all

## Part 8: Screenshots for Documentation

Capture these screenshots for your README:

### 8.1 GitHub Actions
- ✅ Successful workflow run (both jobs green)
- ✅ Build job details
- ✅ Deploy job details

### 8.2 Docker Hub
- ✅ Your Docker Hub repositories page showing both images
- ✅ Tags for each repository (should show `latest`)

### 8.3 Application UI
- ✅ Home page with empty tutorials list
- ✅ Add tutorial form
- ✅ Tutorials list with data
- ✅ Edit tutorial page
- ✅ Search functionality

### 8.4 Infrastructure
- ✅ EC2 instance details page
- ✅ EC2 security groups configuration
- ✅ SSH session showing `docker ps` output
- ✅ Browser showing app running on EC2 IP

### 8.5 Nginx Configuration
- ✅ Nginx configuration file
- ✅ Docker container logs showing nginx running

## Part 9: Continuous Deployment

After initial setup, any push to `main` branch will:

1. ✅ Build new Docker images
2. ✅ Push to Docker Hub
3. ✅ SSH into EC2
4. ✅ Pull latest images
5. ✅ Restart containers
6. ✅ Application automatically updated!

## Troubleshooting

### Issue: GitHub Actions fails at "Build and push backend image"

**Solution:**
- Check Docker Hub credentials in GitHub Secrets
- Verify repository names exist on Docker Hub

### Issue: Deployment fails at SSH step

**Solution:**
- Verify EC2_HOST is correct public IP
- Check EC2_SSH_KEY has complete private key content
- Ensure EC2 security group allows SSH from GitHub Actions IPs (or 0.0.0.0/0)

### Issue: Containers not starting on EC2

**Solution:**
```bash
# SSH into EC2
cd ~/crud-dd-task-mean-app
docker-compose logs

# Rebuild if needed
docker-compose down
docker-compose pull
docker-compose up -d
```

### Issue: Application not accessible

**Solution:**
- Check EC2 security group allows HTTP (port 80)
- Verify nginx container is running: `docker ps | grep nginx`
- Check nginx logs: `docker-compose logs nginx`

### Issue: Backend cannot connect to MongoDB

**Solution:**
```bash
# Check MongoDB is running
docker ps | grep mongodb

# Check backend logs
docker-compose logs backend

# Should see "Connected to the database!"
```

## Cleanup (Optional)

To save costs, when done testing:

### Stop containers but keep data:
```bash
docker-compose down
```

### Stop and remove everything:
```bash
docker-compose down -v
```

### Terminate EC2 instance:
1. Go to EC2 Console
2. Select instance
3. Instance State → Terminate instance

---

**🎉 Congratulations!** You've successfully deployed a full-stack MEAN application with Docker and CI/CD!
